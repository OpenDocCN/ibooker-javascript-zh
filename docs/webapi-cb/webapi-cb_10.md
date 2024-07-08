# 第十章：文件处理

# 引言

读写文件是许多应用程序的一部分。过去，无法在浏览器内直接处理本地文件。要读取数据，您需要将文件上传到后端服务器，服务器处理后返回数据给浏览器。

要写入数据，服务器将发送可下载的文件。没有浏览器插件，无法直接处理文件。

如今，浏览器对于读写文件有了一流的支持。`file` 输入类型打开文件选择器并提供有关所选文件的数据。您还可以限制支持的文件类型为特定扩展名或 MIME 类型。从这里，File API 可以将文件内容读取到内存中。

更进一步，文件系统 API 允许 JavaScript 代码直接与本地文件系统交互，无需首先选择文件输入（尽管根据设置，用户可能需要授予权限！）。

您可以使用这些 API 创建文本编辑器、图像查看器、音频或视频播放器等工具。

# 从文件加载文本

## 问题

您想从用户的本地文件系统加载一些文本数据。

## 解决方案

使用 `<input type="file">` 选择文件（见 Example 10-1）。

##### Example 10-1\. 文件输入

```
<input type="file" id="select-file">
```

当您点击文件输入时，浏览器将显示一个对话框，您可以在其中浏览本地系统中的文件和文件夹。显示的确切对话框将取决于浏览器和操作系统版本。导航到并选择所需的文件。选择文件后，像 Example 10-2 中显示的那样使用 `FileReader` 读取文件的文本内容。

##### Example 10-2\. 从文件加载纯文本

```
/**
 * Reads the text content of a file.
 * @param file The File object containing the data to be read
 * @param onSuccess A function to call when the data is available
 */
function readFileContent(file, onSuccess) {
  const reader = new FileReader();

  // When the content is loaded, the reader will emit a
  // 'load' event.
  reader.addEventListener('load', event => {
    onSuccess(event.target.result);
  });

  // Always handle errors!
  reader.addEventListener('error', event => {
    console.error('Error reading file:', event);
  });

  // Start the file read operation.
  reader.readAsText(file);
}

const fileInput = document.querySelector('#select-file');

// The input fires a 'change' event when a file is selected.
fileInput.addEventListener('change', event => {
  // This is an array, because a file input can be used to select
  // multiple files. Here, there's only once file selected.
  // This is using array destructuring syntax to get the first file.
  const [file] = fileInput.files;

  readFileContent(file, content => {
    // The file's text content is now available.
    // Imagine you have a textarea element you want to set the text in.
    const textArea = document.querySelector('.file-content-textarea');
    textArea.textContent = content;
  });
});
```

## 讨论

`FileReader` 是一个异步读取文件内容的对象。它可以根据文件类型以不同的方式读取文件内容。Example 10-2 使用 `readAsText` 方法，以纯文本形式检索文件内容。

如果您有一个二进制文件，比如 ZIP 归档或图像文件，可以使用 `readAsBinaryString`。图像可以使用 `readAsDataURL` 读取为包含 Base64 编码图像数据的数据 URL，您将在 “加载图像作为数据 URL” 中看到。

此 API 基于事件，因此 `readFileContent` 函数接受一个回调函数，在内容准备好时调用该函数。

您还可以将其包装为 `Promise` 以创建基于 `Promise` 的 API，就像 Example 10-3 中显示的那样。

##### Example 10-3\. 使用 Promise 包装的 `readFileContent` 函数

```
function readFileContent(file) {
  const reader = new FileReader();

  return new Promise((resolve, reject) => {
    reader.addEventListener('load', event => {
      resolve(event.target.result);
    });

    reader.addEventListener('error', reject);

    reader.readAsText(file);
  });
}

try {
  const content = await readFileContent(inputFile);
  const textArea = document.querySelector('.file-content-textarea');
  textArea.textContent = content;
} catch (error) {
  console.error('Error reading file content:', error);
}
```

一旦获取文本内容，您可以通过几种方式将其添加到页面中。您可以将其设置为 DOM 节点的 `textContent`，甚至可以将其加载到 `textarea` 中以进行内容编辑。

# 加载图像作为数据 URL

## 问题

您希望用户选择一个本地图像文件，然后在页面上显示该图像。

## 解决方案

使用`FileReader`的`readAsDataURL`方法获取 Base64 编码的数据 URL，然后将其设置为`img`标签的`src`属性（参见示例 10-4 和 10-5）。

##### 示例 10-4\. 文件输入和图像占位符

```
<input
  type="file"
  id="select-file"
  accept="image/*" ![1](img/1.png)
>
<img id="placeholder-image">
```

![1](img/#co_working_with_files_CO1-1)

限制文件选择器仅允许选择图像。这里使用通配符模式，但您也可以指定确切的 MIME 类型，如`image/png`。

##### 示例 10-5\. 将图像加载到页面中

```
/**
 * Loads and shows an image from a file.
 * @param file The File object containing the image data
 * @param imageElement A placeholder Image element that will
 *                     show the image data
 */
function showImageFile(file, imageElement) {
  const reader = new FileReader();

  reader.addEventListener('load', event => {
    // Set the data URL directly as the image's
    // src attribute to load the image.
    imageElement.src = event.target.result;
  });

  reader.addEventListener('error', event => {
    console.log('error', event);
  });

  reader.readAsDataURL(file);
}

const fileInput = document.querySelector('#select-file');
fileInput.addEventListener('change', event => {
  showImageFile(
    fileInput.files[0],
    document.querySelector('#placeholder-image')
  );
});
```

## 讨论

数据 URL 具有`data` URL 方案。它指定数据的 MIME 类型，然后图像数据以 Base64 编码格式包含在内：

```
data:image/png;base64,UHJldGVuZCB0aGlzIGlzIGltYWdlIGRhdGE=
```

当`FileReader`返回以数据 URL 编码的图像时，将该数据 URL 设置为图像元素的`src`属性。这将在页面中呈现图像。

需要注意的是，所有这些操作都是在用户的浏览器本地进行的。没有任何内容被上传到远程服务器，因为 File API 在本地文件系统上工作。

在第四章的“使用 Fetch API 上传文件”显示了使用`<input type="file">`将文件数据上传到远程服务器的示例，尽管这里使用的是 FormData API 而不是 File API。

关于数据 URL 和 Base64 编码的更多详细信息，请参阅[MDN 上的这篇文章](https://oreil.ly/kMtDy)。

# 加载视频作为对象 URL

## 问题

您希望用户选择一个视频文件，然后在浏览器中播放它。

## 解决方案

为`File`对象创建对象 URL，并将其设置为`<video>`元素的`src`属性。

首先，您需要一个`<video>`元素和一个`<input type="file">`来选择视频文件（请参见示例 10-6）。

##### 示例 10-6\. 视频播放器标记

```
<input
  id="file-upload"
  type="file"
  accept="video/*" ![1](img/1.png)
>

<video
  id="video-player"
  controls ![2](img/2.png)
>
```

![1](img/#co_working_with_files_CO2-1)

仅允许选择视频文件

![2](img/#co_working_with_files_CO2-2)

告诉浏览器包括播放控件

接下来，监听文件输入的`change`事件并创建一个对象 URL，如示例 10-7 所示。

##### 示例 10-7\. 播放视频文件

```
const fileInput = document.querySelector('#file-upload');
const video = document.querySelector('#video-player');

fileInput.addEventListener('change', event => {
  const [file] = fileInput.files;

  // File extends from Blob, which can be passed to
  // createObjectURL.
  const objectUrl = URL.createObjectURL(file);

  // The <video> element can take the object URL to load the video.
  video.src = objectUrl;
});
```

## 讨论

对象 URL 是指向文件内容的特殊 URL。您可以不使用`FileReader`来实现这一点，因为文件本身具有`createObjectURL`方法。此 URL 可以传递给`<video>`元素。

# 拖放加载图像

## 问题

您希望能够将图像文件拖放到浏览器窗口中，并在放置时在页面上显示该图像。

## 解决方案

定义一个用作拖放区域的元素和一个占位符图像元素（参见示例 10-8）。

##### 示例 10-8\. 拖放目标和图像元素

```
<label id="drop-target">
  <div>Drag and drop an image here</div>
  <input type="file" id="file-input">
</label>
<img id="placeholder">
```

请注意，此示例仍然包括文件`input`。这是为了使使用辅助技术的用户也可以上传图像，而无需尝试拖放操作。因为拖放目标是一个包含文件输入的标签，您可以在拖放目标的任何位置点击以打开文件选择器。

首先，创建一个函数，接收图片文件并将其作为数据 URL 读取（见 示例 10-9）。

##### 示例 10-9\. 读取拖放的文件

```
function showDroppedFile(file) {
  // Read the file data and insert the loaded image
  // into the page.
  const reader = new FileReader();
  reader.addEventListener('load', event => {
    const image = document.querySelector('#placeholder');
    image.src = event.target.result;
  });

  reader.readAsDataURL(file);
}
```

接下来，为 `dragover` 和 `drop` 事件创建处理函数。这些事件附加到拖放目标元素（见 示例 10-10）。

##### 示例 10-10\. 添加拖放代码

```
const target = document.querySelector('#drop-target');
target.addEventListener('drop', event => {
  // Cancel the drop event. Otherwise, the browser will leave the page
  // and navigate to the file directly.
  event.preventDefault();

  // Get the selected file data. dataTransfer.items is a
  // DataTransferItemList. Each item in the list, a DataTransferItem, has data
  // about an item being dropped. As this example only deals with a single file, it
  // gets the first item in the list.
  const [item] = event.dataTransfer.items;

  // Get the dropped data as a File object.
  const file = item.getAsFile();

  // Only proceed if an image file was dropped.
  if (file.type.startsWith('image/')) {
    showDroppedFile(file);
  }
});

// You need to cancel the dragover event as well to prevent the
// file from replacing the full page content.
target.addEventListener('dragover', event => {
  event.preventDefault();
});
```

最后，确保连接后备文件输入。你只需获取选择的文件，然后将其传递给 `showDroppedFile` 方法以获得相同的结果（见 示例 10-11）。

##### 示例 10-11\. 处理文件输入

```
const fileInput = document.querySelector('#file-input');
fileInput.addEventListener('change', () => {
  const [file] = fileInput.files;
  showDroppedFile(file);
});
```

## 讨论

默认情况下，当你将一个图片拖放到页面上时，浏览器会离开当前页面。URL 会变成文件路径，并且图片会显示在浏览器窗口中。在这个示例中，你希望将图片数据加载到一个 `<img>` 元素中，并保持在当前页面。

为了阻止默认行为，拖放处理程序在 drop 事件上调用 `preventDefault`。为了完全阻止这种行为，你还需要在 `dragover` 事件上调用 `preventDefault`，这就是为什么你需要第二个事件侦听器。这样可以确保元素实际上可以接收 `drop` 事件。

# 检查和请求权限

## 问题

你需要检查——并在必要时请求——访问本地文件系统上的文件权限。

## 解决方案

显示文件选择器，当选择文件时，调用 `queryPermission` 检查现有权限。如果权限检查返回 `prompt`，则调用 `requestPermission` 显示权限请求（见 示例 10-12）。

##### 示例 10-12\. 选择和检查文件权限

```
/**
 * Selects a file, then checks permissions, showing a request if necessary,
 * for a file.
 * @return true if the file can be written to, false otherwise
 */
async function canAccessFile() {
  if ('showOpenFilePicker' in window) {
    // showOpenFilePicker can select multiple files, just
    // get the first one (with array destructuring).
    const [file] = window.showOpenFilePicker();

    let result = await file.queryPermission({ mode: 'readwrite' });
    if (result === 'prompt') {
      result = await file.requestPermission({ mode: 'readwrite' });
    }

    return result === 'granted';
  }

  // If you get here, it means the API isn't supported.
  return false;
}
```

###### 注意

这个 API 可能还不被所有浏览器支持。查看 [CanIUse](https://oreil.ly/AfNpL) 获取最新的兼容性数据。

## 讨论

`queryPermission` 函数返回 `granted`（已授予权限）、`denied`（访问被拒绝）或 `prompt`（需要请求权限）。

请求模式为 `readwrite`，这意味着如果你授予权限，浏览器可以写入你的本地文件系统。因此，从安全和隐私角度来看，权限检查非常重要。

`queryPermission` 只检查权限而不显示提示。如果返回 `prompt`，则可以调用 `requestPermission` 在浏览器中显示权限请求。如果任一调用返回 `granted`，则认为文件可写。

# 将 API 数据导出到文件

## 问题

你正在从一个 API 请求 JSON 数据，并且你希望给用户一个选项来下载原始的 JSON 数据。

## 解决方案

让用户选择一个输出文件，然后将 JSON 数据写入到本地文件系统。

###### 注意

这个 API 可能还不被所有浏览器支持。查看 [CanIUse](https://oreil.ly/tsT_j) 获取最新的兼容性数据。

首先，定义一个辅助函数，显示文件选择器并返回所选择的文件（见 示例 10-13）。

##### 示例 10-13\. 选择输出文件

```
/**
 * Shows a save file picker and returns the selected file handle.
 * @returns a file handle to the selected file, or null if the user clicked Cancel.
 */
async function selectOutputFile() {
  // Check to make sure the API is supported in this browser.
  if (!('showSaveFilePicker' in window)) {
    return null;
  }

  try {
    return window.showSaveFilePicker({
      // The default name for the output file
      suggestedName: 'users.json',

      // Limit the available file extensions.
      types: [
        { description: "JSON", accept: { "application/json": [".json"] } }
      ]
    });
  } catch (error) {
    // If the user clicks Cancel, an exception is thrown. In this case,
    // return null to indicate no file was selected.
    return null;
  }
}
```

接下来，定义一个使用这个帮助函数的函数，并执行实际的导出操作（参见 示例 10-14）。

##### 示例 10-14\. 将数据导出到本地文件

```
async function exportData(data) {
  // Use the helper function defined previously.
  const outputFile = await selectOutputFile();

  // Only proceed if an output file was actually selected.
  if (outputFile) {
    try {
      // Prepare a writable stream, which is used to save the file
      // to disk.
      const stream = await outputFile.createWritable();

      // Write the JSON t the stream in a human-readable format.
      await stream.write(JSON.stringify(userList, null, 2));
      await stream.close();

      // Show a success message.
      document.querySelector('#export-success').classList.remove('d-none');
    } catch (error) {
      console.error(error);
    }
  }
}
```

## 讨论

这是一个允许用户从你的应用中备份或导出数据的好方法。一些法规，比如欧盟的《通用数据保护条例》（GDPR），要求你让用户下载他们的数据。

在这种情况下，文本数据被写入流中，流的类型是 `FileSystem` 的 `WritableFileStream`。这些流也支持写入 `ArrayBuffer`、`TypedArray`、`DataView` 和 `Blob` 对象。

为了创建写入文件的文本，`exportData` 调用 `JSON.stringify` 函数并传入一些额外的参数。第二个 `null` 参数是 `replacer` 函数，你可以在 第二章 中看到它。这个参数必须提供以便传入第三个参数，用来指定缩进空格的数量，从而创建一个更易读的输出格式。

在撰写本文时，这个 API 仍然被视为实验性质。在它有更好的浏览器支持之前，你应该避免在生产应用中使用它。

# 使用下载链接导出 API 数据

## 问题

你想提供导出功能，但又不想担心文件系统的权限问题，就像在 “将 API 数据导出到文件” 中描述的那样。

## 解决方案

将 API 数据放入 `Blob` 对象中，并创建一个对象 URL 以设置为链接的 `href` 属性。然后你可以通过普通的浏览器文件下载导出数据，而无需文件系统的权限。

首先，在页面上添加一个占位符链接，这个链接会成为导出链接（参见 示例 10-15）。

##### 示例 10-15\. 导出链接的占位符

```
<a id="export-link" download="users.json">Export User Data</a> ![1](img/1.png)
```

![1](img/#co_working_with_files_CO3-1)

`download` 属性提供了一个默认的文件名用于下载。

在从 API 获取数据并在 UI 中渲染后，创建 `Blob` 和对象 URL（参见 示例 10-16）。

##### 示例 10-16\. 准备导出链接

```
const exportLink = document.querySelector('#export-link');

async function getUserData() {
  const response = await fetch('/api/users');
  const users = await response.json();

  // Render the user data in the UI, assuming that you
  // have a renderUsers function somewhere that does this.
  renderUsers(users);

  // Clean up the previous export data, if it exists.
  const currentUrl = exportLink.href;
  if (currentUrl) {
    URL.revokeObjectURL(currentUrl);
  }

  // Need a Blob for creating an object URL
  const blob = new Blob([JSON.stringify(userList, null, 2)], {
    type: 'application/json'
  });

  // The object URL links to the Blob contents—set this in the link.
  const url = URL.createObjectURL(blob);
  exportLink.href = url;
}
```

## 讨论

这种导出方法不需要特殊权限。当链接被点击并且对象 URL 已经设置时，它会将 `Blob` 的内容作为一个文件下载下来，文件名建议为 *users.json*。

一个 `Blob` 是一个特殊的对象，用来保存一些数据片段。通常这些数据是二进制的，比如文件或者图片，但你也可以用字符串内容创建一个 `Blob`，这就是这个示例要做的事情。

`Blob` 存储在内存中，并且创建的对象 URL 链接到它。一旦链接元素的对象 URL 设置好了，它就变成了一个导出下载链接。当链接被点击时，对象 URL 返回原始字符串数据。由于链接有一个 `download` 属性，它会被下载到本地文件。

为了防止内存泄漏，通过调用 `URL.revokeObjectURL` 并将对象 URL 作为其参数来清除旧的 URL。当你不再需要对象 URL 时（例如用户下载文件或离开页面之前），可以执行此操作。

# 使用拖放上传文件

## 问题

允许用户拖放文件（例如图片），然后将该文件上传到远程服务。

## 解决方案

将接收到的 `File` 对象传递给 `drop` 事件的处理函数中的 Fetch API（参见 示例 10-17）。

##### 示例 10-17\. 上载拖放的文件

```
const target = document.querySelector('.drop-target');

target.addEventListener('drop', event => {
  // Cancel the drop event. Otherwise, the browser will leave the page
  // and navigate to the file directly.
  event.preventDefault();

  // Get the selected file data.
  const [item] = event.dataTransfer.items;
  const file = item.getAsFile();

  if (file.type.startsWith('image/')) {
    fetch('/api/uploadFile', {
      method: 'POST',
      body: file
    });
  }
});

// You need to cancel the dragover event as well, to prevent the
// file from replacing the full page content.
target.addEventListener('dragover', event => {
  event.preventDefault();
});
```

## 讨论

当在数据传输对象上调用 `getAsFile` 时，会得到一个 `File` 对象。`File` 是 `Blob` 的扩展，因此可以使用 Fetch API 将文件（`Blob`）内容发送到远程服务器。

此示例检查上传文件的 MIME 类型，仅当其为图片文件时才会上传。
