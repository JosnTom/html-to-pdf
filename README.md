# html-to-pdf


#                        关于AspNetCoreWebApi 把 Html 导出为 PDF



## 作者开发环境为 Core 3.1 , iis 部署环境为 Core6.1 



## 1.下载对应【Net】服务器开发版本，建议安装最高版本

​    地址：https://dotnet.microsoft.com/zh-cn/download/dotnet/6.0

![image-20220628150420718](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220628150420718.png)



## 2.安装google chrome 浏览器 （必须为google浏览器）

  地址： https://www.google.cn/intl/zh-cn/chrome/  



##  3. 查找google 用户配置文件地址

   参考目录：C:\Users\yyadmin\AppData\Local\Google\Chrome\User Data    

   您应当在安装google chrome 后最终地址如下：  【yyadmin 】为您的电脑名称

![image-20220628153242754](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220628153242754.png)

## 4.部署服务在IIS7.5下测试 ，其它未测试

​      程序池部署如下：

   ![image-20220628150838265](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220628150838265.png)

![image-20220628150853861](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220628150853861.png)



## 5.0  Ajax  Post 请求配置参数如下：

 ``` c#
  public class ViewOptionModel
    {
        /// <summary>
        /// 默认计算机名称  存放google配置文件
        /// </summary>
        public string User { get; set; }
        /// <summary>
        /// url域名地址
        /// </summary>
        public string Url { get; set; }

        /// <summary>
        /// 模板名称
        /// </summary>
        public string Template { get; set; }
        /// <summary>
        /// 文件名称
        /// </summary>
        public string FileName { get; set; }

        /// <summary>
        /// 文件夹名称
        /// </summary>
        public string FileRoot { get; set; }

        /// <summary>
        /// 文件内容 html
        /// </summary>
        public string FileContent { get; set; }

        /// <summary>
        /// 脚底标题
        /// </summary>
        public bool DisplayHeaderFooter { get; set; } = false;

        /// <summary>
        /// 背景
        /// </summary>
        public bool PrintBackground { get; set; } = true;

        /// <summary>
        /// css 样式
        /// </summary>
        public bool PreferCSSPageSize { get; set; } = true;

        /// <summary>
        /// 页码
        /// </summary>
        public bool IgnoreInvalidPageRanges { get; set; } = true;

        /// <summary>
        /// 打印宽度  单位为英寸 A4
        /// </summary>
        public double PaperHeight { get; set; } = 8.5;

        /// <summary>
        /// 打印高度
        /// </summary>
        public double PaperWidth { get; set; } = 8.5;

     
    }


 ```



## 6.0  JavaScript 配置参考如下：

 ``` javascript
  
// 建议放在html代码最外层
 $(document).ready(function () {
        // 为 FileContent 放在底部 减少数据发送量
        function get_page_tip() {
            this.isMobile = function isMobile() {
                var userAgentInfo = navigator.userAgent;
                var mobileAgents = ["Android", "iPhone", "SymbianOS", "Windows Phone", "iPad", "iPod"];
                var mobile_flag = false;
                //根据userAgent判断是否是手机
                for (var v = 0; v < mobileAgents.length; v++) {
                    if (userAgentInfo.indexOf(mobileAgents[v]) > 0) {
                        mobile_flag = true;
                        break;
                    }
                }
                var screen_width = window.screen.width;
                var screen_height = window.screen.height;
                //根据屏幕分辨率判断是否是手机
                if (screen_width < 500 && screen_height < 800) {
                    mobile_flag = true;
                }
                return mobile_flag;
            },
                // 成功
                this.SUCCESS = function (content, textColor) {
                    if (content == '' || content == null || content == undefined) {
                        content = '您的操作已成功';
                    }
                    var px = this.isMobile() ? "70%" : "20%";
                    var ding_html =
                            `<div class="message-main" style="width:${px};border:1px solid rgb(249,249,249);background-color:#dee7ed;margin:0 auto;border-radius:6px;padding-top:10px;z-index:9999;position:fixed;left:0;right:0;top:20%;">`;
                        ding_html +=
                            `<div class="message-content" style="margin:5px;text-align:center;height:50px;line-height:50px; color:${textColor}">${content} </div>`;
                        ding_html +=
                            `<div class="message-footer" style="margin:5px;display:flex;flex-flow:row nowrap;justify-content:space-between;border-top:1px solid rgb(240,240,240);padding-top:5px;">`;
                        ding_html +=
                            `<div class="footer-left" id='ding-left' style="width: 50%;text-align: center; color: #235FF5;border-right: 1px solid rgb(240, 240, 240);">取消</div>`;
                        ding_html +=
                            `<div class="footer-right" id='ding-right' style="width: 50%;text-align: center; color: #235FF5;">确定</div>`;
                    ding_html += `</div>`;
                    ding_html += `</div>`;

                    var colse = document.getElementById('app-ding');
                    if (colse != null) {
                        colse.remove();
                    }
                    var appding = document.createElement("div");
                    appding.id = 'app-ding';
                    appding.innerHTML = ding_html;
                    document.body.appendChild(appding);

                    document.getElementById('ding-right').onclick = function () {
                        appding.style.display = 'none';
                    }
                    document.getElementById('ding-left').onclick = function () {
                        appding.style.display = 'none';
                    }
                    return this;
                }
            // 错误提示
            this.ERROR = function (content, textColor) {

            }
        }
       //触发打印事件  这里是核心，请看这里
        $('#submit').click(function () {
            new get_page_tip().SUCCESS("正在导出pdf,请稍后", "");
            var html = $('html').html();
            var FileRoot = "ceshi"; //文件存放目录
            var settings = {
                "url": "/pdf/api/school/activepdf",
                "method": "POST",
                "timeout": 0,
                "headers": {
                    "Content-Type": "application/json"
                },
                "data": JSON.stringify({
                     //打印内容  Url 优先级高于 FileContent
                    "Url": "http://school.yuysoft.com/pdf/School/ceshi/index.html", 
                    "FileContent":html,
                    "User": "yyadmin",
                    "FileName": "dingxue",  //文件名称
                    "FileRoot": FileRoot, //生成pdf目录
                    "PaperHeight": 15, //打印高度  默认英寸
                    "PaperWidth": 15 //打印宽度 默认英寸
                }),
            };

            $.ajax(settings).done(function (response) {
                if (response.success) {
                    $('#app-ding').hide();
                    //下载文件地址
                    window.location.href = `/pdf/api/school/${FileRoot}/${response.data}`;
                } else {
                    alert(response.msg);
                }
            });
        });
    }); 

 ```



##  7.0 更多配置请参考 压缩包源代码  或  原作者源代码：

  地址： https://github.com/Sicos1977/ChromeHtmlToPdf

  请不要忘记start一下

## 8.0 注意事项，在打印网页时 图片的地址要为相对域名地址：或绝对路径

  参考：1.  https://www.baidu/img/img.png

​             2.  /img/img.png
