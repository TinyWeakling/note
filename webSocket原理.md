#webSocket原理

**安装ws**

*npm install ws*

##服务端
    var webSocketServer = require('ws').Server;//注意Server首字母大写
    var wss = new webSocketServer({port:3001});//指定服务端口
    wss.on('connection',function(ws){
        ws.on('message',function(msg){
            console.log('这里是客户端发送来的消息:'+msg);
            ws.send('你好！');//发送给客户端的消息
        })
    });
    
##客户端

    var webSocket=require('ws');
    var ws = new webSocket('ws://localhost:3001');
    ws.on('open',function(){
        ws.send('第一次链接！');
    });
    ws.on('message',function(msg){
        console.log('收到服务器的消息：'+msg);
    });
    
##浏览器端

    <script>
        var ws = new WebSocket('ws://localhost:3001');//WebSocket为H5的新特性
        ws.onopen=function(){
            ws.send('连接成功！');
        }
        ws.onmessage=function(evt){
            document.write(evt.data);
        }
    </script>