element-ui的上传组件其实可以分为两部分，一部分是真正的上传，它负责完成上传功能，另一部分是上传列表，负责展示上传进度、上传结果。

upload 这个组件，element-ui实现的有点问题，当遇到多个上传时会有问题。这里的难点在于如何同步本地上传列表和上层组件的的列表，因为牵扯到了多个不确定时序的异步。相比较之下[百度的veui](https://baidu-design.github.io/development/components/uploader)解决了这个问题。说是上传组件，如果把它看成表单的一部分，我们所关心的是上传后后端所返回的值，而上传本身只是获取这个值的方式。



## 上传

上传组件采用了render函数的形式，而不是模板，我先试着还原成模板的形式：

```html
<template>
    <div @click="handleClick">
        <template v-if="drag">
            <upload-dragger
                @file="uploadFiles"
            >
                <slot></slot>
            </upload-dragger>
        </template>
        <template v-else>
            <slot></slot>
        </template>
        <input 
            type="file" 
            class="el-upload__input"
            ref="input"
            @change="handleChange"
            :accept="accept"
            :multiple="multiple"
        />
    </div>
</template>
```

类型是file的input元素被CSS设置为不显示，外层div监听click事件，然后利用类型为file的input呼出文件选择对话框：

```javascript
handleClick() {
    if (!this.disabled) {
        this.$refs.input.value = null;
        // 触发点击事件，调出文件选择对话框
        this.$refs.input.click();
    }
}
```

选择文件后，会触发input的change事件，我们就可以获得选择的文件：

```javascript
handleChange(ev) {
    // 获得要上传的文件
    const files = ev.target.files;
    if (!files) return;

    this.uploadFiles(files);
},
```

获得上传的文件后，我们就可以上传文件了：

```javascript
// 上传一组文件
uploadFiles(files) {
    let postFiles = Array.prototype.slice.call(files);
    // 校验限定只上传一个的情况
    if (!this.multiple) { postFiles = postFiles.slice(0, 1); }

    if (postFiles.length === 0) { return; }

    postFiles.forEach(rawFile => {
        // onStart是父组件传过来的方法，用来对file做一些操作
        this.onStart(rawFile);

        // 自动上传
        if (this.autoUpload) this.upload(rawFile);
    });
},
// 上传一个文件
upload(rawFile, file) {
    this.$refs.input.value = null;

    if (!this.beforeUpload) {
        return this.post(rawFile);
    }

    // 处理beforeUpload钩子
    // beforeUpload可以返回一个promise，也可以返回布尔值表示能否上传
    const before = this.beforeUpload(rawFile);
    if (before && before.then) {
        before.then(processedFile => {
            if (Object.prototype.toString.call(processedFile) === '[object File]') {
                this.post(processedFile);
            } else {
                this.post(rawFile);
            }
        }, () => {
            this.onRemove(rawFile, true);
        });
    } else if (before !== false) {
        this.post(rawFile);
    } else {
        this.onRemove(rawFile, true);
    }
},
post(rawFile) {
    const { uid } = rawFile;
    // 上传的配置项
    const options = {
        headers: this.headers,
        withCredentials: this.withCredentials,
        file: rawFile,
        data: this.data,
        filename: this.name,
        action: this.action,
        onProgress: e => {
            this.onProgress(e, rawFile);
        },
        onSuccess: res => {
            this.onSuccess(res, rawFile);
            // 请求成功，不需要放弃上传了
            delete this.reqs[uid];
        },
        onError: err => {
            this.onError(err, rawFile);
            // 请求失败，也不用放弃上传
            delete this.reqs[uid];
        }
    };
    // 到这才是真正的上传
    // httpRequest有默认的方法，也允许开发者定制上传方法
    const req = this.httpRequest(options);

    // 缓存上传请求，放弃上传时用
    this.reqs[uid] = req;

    // 自定义上传的情况，考虑promise
    if (req && req.then) {
        req.then(options.onSuccess, options.onError);
    }
},
```

element-ui提供了一个默认的httpRequest方法，这个方法也允许开发者根据业务进行定制，默认的方法本身就是对于ajax操作的一个基本封装：

```javascript
export default function upload(option) {
    if (typeof XMLHttpRequest === 'undefined') {
        return;
    }

    const xhr = new XMLHttpRequest();
    const action = option.action;

    // 上传进度，提供给上传结果列表使用
    if (xhr.upload) {
        xhr.upload.onprogress = function progress(e) {
            if (e.total > 0) {
                e.percent = e.loaded / e.total * 100;
            }
            option.onProgress(e);
        };
    }

    const formData = new FormData();

    if (option.data) {
        Object.keys(option.data).map(key => {
            formData.append(key, option.data[key]);
        });
    }

    formData.append(option.filename, option.file);

    xhr.onerror = function error(e) {
        option.onError(e);
    };

    xhr.onload = function onload() {
        // 校验返回状态码
        if (xhr.status < 200 || xhr.status >= 300) {
            return option.onError(getError(action, option, xhr));
        }

        option.onSuccess(getBody(xhr));
    };

    xhr.open('post', action, true);

    // CORS跨域的cookie设置
    if (option.withCredentials && 'withCredentials' in xhr) {
        xhr.withCredentials = true;
    }

    // 设定请求头
    const headers = option.headers || {};
    for (let item in headers) {
        if (headers.hasOwnProperty(item) && headers[item] !== null) {
            xhr.setRequestHeader(item, headers[item]);
        }
    }
    xhr.send(formData);
    return xhr;
}
```

对于上传组件，有许多通过props传递过来的方法，比如说上面提到的beforeUpload和httpRequest，这些方法允许我们细粒度得调整组件内部运行状态。


## 拖拽

拖拽组件其实只是一个很小的附属组件，它允许以拖拽形式选择要上传的文件，而不是仅仅是弹出一个选择文件对话框。之所以把它单独拿出来说是因为他用了HTML5的拖拽API。

```html
<template>
  <div
    class="el-upload-dragger"
    :class="{
      'is-dragover': dragover
    }"
    @drop.prevent="onDrop"
    @dragover.prevent="onDragover"
    @dragleave.prevent="dragover = false"
  >
    <slot></slot>
  </div>
</template>
```

当我们把要上传的文件拖拽进来的时候，就可以通过拖拽API获取文件列表：

```javascript
onDrop(e) {
    if (!this.disabled) {
        this.dragover = false;
        this.$emit('file', e.dataTransfer.files);
    }
}
```

拖拽组件把上传文件列表传给上传组件，由上传组件完成上传任务。