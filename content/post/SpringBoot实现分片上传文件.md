---
title: SpringBoot实现分片上传文件
date: 2023-10-21 11:56:11
tags: [SpringBoot, JavaScript]
categories: [技术分享]
---

大家好，这里是胡编乱码，欢迎大家收看本期内容，本期内容将介绍文件分片上传的技术和具体实现。

本次演示的 Demo 前端比较简单，所以使用纯 html/js/css 来实现，前端的网络请求我将使用 Axios.js 来完成。服务端的话我将使用 SpringBoot 来开发，那么废话不多说，我们直接开始。

### 实现原理

实现分片上传的原理大概分为两步，分别对应前后端：

1. 前端将文件进行分片、计算 MD5 码和将分片上传。
  
2. 服务端将接收到的分片文件根据 MD5 码拼接成一个完整的文件。
  

![实现原理](/images/20231021/SpringBoot实现分片上传.jpg)

### 服务端实现

首先我们创建一个 SpringBoot 项目，创建方式可以通过官网提供的创建项目页面进行创建，网站地址为：[https://start.spring.io/](https://start.spring.io/)。下图是我选择的创建配置，依赖的添加上Web依赖即可：

![](/images/20231021/2023-10-20-21-34-58-image.png)

然后我们点击 Generate 按钮将我们的项目工程下载下来，解压后通过打开项目进行开发就可以。接下来我们编写分片上传接口。首先我们确定一下上传的参数有哪些？

1. 文件内容
  
2. 文件名称
  
3. 文件MD5码
  
4. 文件分片总数
  
5. 当前分片索引
  

根据上面5个参数我们就可以完成分片上传服务端的实现，其中文件名称是用来在拼接为完整的文件后对文件进行命名，MD5是用来识别哪些需要拼接在一起，文件分片总数是用来判断文件是否已经上传完成，当前分片索引是用来确定当前接受到的文件分片应该拼接到最终文件的哪里，下面是实现代码

```java
package vip.huhailong.uploadslice.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.RandomAccessFile;
import java.util.Map;
import java.util.Objects;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author 胡编乱码
 * 文件接口
 */
@RestController
@RequestMapping("/file")
public class FileController {

  private static  final Logger logger = LoggerFactory.getLogger(FileController.class);

  @Value("${slice.saveDir}")
  private String saveDir;

  private static final Map<String, Integer> fileCountMap = new ConcurrentHashMap<>();

  /**
   * 分片上传接口
   * @param file 分片文件
   * @param fileName 文件名称
   * @param fileMd5 文件MD5值
   * @param chunkCount 分片总数量
   * @param currentIndex 当前分片索引
   */
  @PostMapping("/uploadBySlice")
  public float upload(MultipartFile file, String fileName, String fileMd5, Integer chunkCount, Integer currentIndex){
    String tempFilePath = saveDir + File.separator + fileMd5 + ".tmp";
    checkFilePath();
    try(RandomAccessFile accessFile = new RandomAccessFile(tempFilePath,"rw")){
      accessFile.seek(currentIndex);
      accessFile.write(file.getBytes());
      fileCountMap.put(fileMd5, fileCountMap.getOrDefault(fileMd5,0)+1);
      if(Objects.equals(fileCountMap.get(fileMd5), chunkCount)){
	  	accessFile.close();
        File tempFile = new File(tempFilePath);
        File finallyFile = new File(saveDir + File.separator + fileName);
        tempFile.renameTo(finallyFile);
        tempFile.delete();
        logger.info("上传完成：{}",saveDir+File.separator+fileName);
        fileCountMap.remove(fileMd5);
        return 1.0f;
      }else{
        logger.info("{}-{}件上传中",fileMd5,currentIndex);
        return (float) fileCountMap.get(fileMd5) / chunkCount;
      }
    }catch (Exception e){
      logger.error(e.getMessage());
      throw new RuntimeException(e);
    }
  }

  private void checkFilePath(){
    File file = new File(saveDir);
    if(!file.exists()){
      file.mkdirs();
    }
  }
}
```

前端代码实现

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文件分片上传</title>
    <script src="axios.min.js"></script>
    <script src="spark-md5.min.js"></script>
    <style>
        html,body{
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100vh;
            background-color: #333;
        }
        .uploadBox{
            width: 600px;
            box-shadow: 2px 3px 10px black;
            border-radius: 4px;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: end;
            justify-content: center;
            background-color: gainsboro;
        }
        .uploadBtn{
            width: 100px;
            height: 20px;
            background-color: darkcyan;
            border-radius: 4px;
            text-align: center;
            color: white;
            display: flex;
            padding: 5px 10px;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            cursor: pointer;
        }
        .uploadBtn:hover{
            opacity: 0.9;
        }
        .fileInfo{
            width: 100%;
            text-align: left;
            display: flex;
            justify-content: center;
        }
        .fileInfo > table {
            border: 1px solid black;
            border-collapse: 0px;
            width: 100%;
            margin-top: 10px;
            padding: 10px;
            border-radius: 4px;
        }
        header{
            position: fixed;
            top: 0;
            height: 40px;
            background-color: #000;
            color: white;
            display: flex;
            align-items: center;
            justify-content: start;
            width: 100%;
            padding: 10px;
        }
    </style>
</head>
<body>
    <header>
        <span style="padding-left: 10px; font-size: 18px; font-weight: bold;">胡编乱码</span>
    </header>
    <div class="uploadBox">
        <div id="upload" class="uploadBtn">上传文件</div>
        <input type="file" id="file" hidden/>
        <div class="fileInfo">
            <table>
                <caption>文件信息</caption>
                <tr><th>文件名称</th><td id="fileName">为选择文件</td></tr>
                <tr><th>文件大小</th><td id="fileSize">0 kb</td></tr>
                <tr><th>分片大小</th><td id="sliceSize">0 kb</td></tr>
                <tr><th>分片数量</th><td id="sliceCount">0</td></tr>
                <tr><th>MD5码</th><td id="fileMd5">-</td></tr>
                <footer>
                    <tr><th>上传状态</th><td id="status"></td></tr>
                </footer>
            </table>
        </div>
        <div style="width: 100%; text-align: left; padding-top: 10px; font-size: 14px; color: gray;">V0.0.1 2023/10/21 15:10</div>

    </div>
    <script>
        document.getElementById("upload").addEventListener('click',()=>{
            document.getElementById("file").click()
        })
        document.getElementById("file").addEventListener("change",(e)=>{
            let file = e.target.files[0];
            document.getElementById("fileName").innerText = file.name;
            document.getElementById("fileSize").innerText = Math.ceil(file.size / 1024) + 'kb';
            sliceFileAndUpload(file, 1000 * 1024);
        })

        function sliceFileAndUpload(file, chunkSize){
            var filePartList = [];
            var blobSlice = File.prototype.slice || File.prototype.mozSlice || File.prototype.webkitSlice;
            var chunkCount = Math.ceil(file.size / chunkSize);  //分片数量
            document.getElementById("sliceSize").innerHTML = chunkSize;
            document.getElementById("sliceCount").innerHTML = chunkCount;
            var currentChunk = 0;   //当前索引
            var spark = new SparkMD5.ArrayBuffer();
            var fileReader = new FileReader();
            fileReader.onload = e => {  //FileReader.onload 回调函数会在每次读区完成后触发
                spark.append(e.target.result);
                currentChunk++;
                if(currentChunk < chunkCount){
                    loadNext()
                    document.getElementById("fileMd5").innerHTML = '计算中……'
                }else{
                    var fileMd5 = spark.end();
                    document.getElementById("fileMd5").innerHTML = fileMd5;
                    var progressEl = document.getElementById("status");
                    filePartList.forEach((item,index)=>{
                        let formData = new FormData();
                        formData.append('file', item.filePart);
                        formData.append('fileMd5', fileMd5);
                        formData.append('chunkCount', chunkCount);
                        formData.append('currentIndex', item.currentIndex);
                        formData.append('fileName', file.name);
                        axios.post('/file/uploadBySlice',formData,{headers:{'Content-Type': 'multipart/form-data'}}).then(res=>{
                            console.log(res.data)
                            if(res.data == 1){
                              progressEl.innerHTML = '上传完成';
                            }else{
                              progressEl.innerHTML = '上传中……';
                            }
                        })
                    })

                }
            }
            fileReader.onerror = () => {
                console.log("文件读取处理发生错误");
            }

            function loadNext() {
                var temp = {};
                var start = currentChunk * chunkSize;
                var end = (start + chunkSize) >= file.size ? file.size : start + chunkSize;
                var filePart = blobSlice.call(file, start, end);    //分隔文件
                temp.currentIndex = start;
                temp.filePart = filePart;
                filePartList.push(temp);    //将分片文件加入到分片文件列表中
                fileReader.readAsArrayBuffer(filePart); //将分片文件使用FileReader读取
            }

            loadNext();
        }
    </script>
</body>
</html>
```

前端代码中主要看 `sliceFileAndUpload`方法的实现，注意分隔的起始和结束位置的确定，具体讲解可看视频中的讲解。

#### 代码仓库地址

- Gitee：[upload-slice: 分片上传文件Demo项目](https://gitee.com/hlovez/upload-slice.git)
  
- GitHub: https://github.com/xiaohu-pro/upload-slice.git
