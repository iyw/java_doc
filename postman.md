postman 自动生成签名
```js
var appkey = KEY; // 自动化测试的key
 //获取当前时间
 function createTime() {
     return (new Date()).valueOf();
 }
 var time = createTime();
 var method = request.method;//提交方式

 delete request.data["sign"];//将sign排除排序
 console.log("request data is : " + request.data);
 var keys = Object.keys(request.data), i, len = keys.length;
 keys.sort();//根据key排序
 // Build the request body string from the Postman request.data object
 var requestBody = "";
 var firstpass = true;
 // 构造数据为 key=param&key=param....字符串
 for(var index in keys){
 if (keys[index] == "sign") {
  continue;
 }
if(!firstpass){
    requestBody += "&";
}
        
if(keys[index]=="create_time"){
    request.data[keys[index]]=time;
    console.log(request.data[keys[index]]);
    }
    requestBody += keys[index] + "=" + request.data[keys[index]];
    firstpass = false;
}
requestBody += '&key=' + appkey;

 //生成密钥
 var md5=CryptoJS.MD5(requestBody, appkey);
 console.log(md5);
 var base64md5 = CryptoJS.enc.Base64.stringify(md5);
 console.log(base64md5);
 if (method.toLowerCase() == 'get') {
      console.log(123);
 base64md5 = encodeURIComponent(base64md5);
}
 console.log("base64md5: " + base64md5);
 //    将变量放入postman 变量中;
 postman.setEnvironmentVariable('sign', base64md5);

```