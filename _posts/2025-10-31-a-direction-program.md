---
layout: post
title: 地图导航项目设计
date: 2025-10-31 05:45 +0000
categories: [Program]
tag: [JavaScript, HTML, CSS, Node.js, Express, Web Design, frontend, backend]
math: true
---

## ***简单网页设计***

## **后端设计**

### **一、所需工具**

#### **1.工具和框架**

由于该项目仅供学习使用，不接入真实服务器，故使用Node.js来部署一个本地HTTP服务器，并用Express框架来处理前端请求，托管静态网页，图片和脚本程序。

``` javascript
const express = require('express'); // 导入Express框架
const path = require('path');// 安全处理和拼接文件路径
const app = express(); // 创建一个express应用对象，
const PORT = 3000; // 设置端口号，3000是惯例设置

app.use(express.json()); // 主要用来处理前端发送的POST请求
app.use(express.static(path.join(__dirname, 'src/public')));
app.use('/res', express.static(path.join(__dirname, 'res')));
```

- 这里path的作用是安全处理和拼接文件路径。在Windows里的文件路径是用'\'，而macOS和Linux里的文件路径是用'/'，因此使用path确保文件路径的一致性，防止因操作系统的不同导致获取错误的文件路径或导致转义；

- express.static() 作用是托管静态文件。当浏览器访问URL时会向后端发送请求，当请求到达服务器时，Express会去我指定的路径去找前端文件，然后直接返回给浏览器，浏览器接收后就可以渲染页面。

- path.join() 作用是生成绝对路径。在这里是将__dirname（当前所在文件路径）和需要托管的文件路径拼接到一起，并防止操作系统不同导致托管错误。

#### **2. WGS84 转 BD09**

```javascript
function wgs84ToBd09(lat, lng) {
    const a = 6378245.0;
    const ee = 0.00669342162296594323;

    const dLat = transformLat(lng - 105.0, lat - 35.0);
    const dLng = transformLng(lng - 105.0, lat - 35.0);
    const radLat = lat / 180.0 * Math.PI;
    const magic = Math.sin(radLat);
    const sqrtMagic = Math.sqrt(1 - ee * magic * magic);
    const mgLat = lat + (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * Math.PI);
    const mgLng = lng + (dLng * 180.0) / (a / sqrtMagic * Math.cos(radLat) * Math.PI);

    const x = mgLng, y = mgLat;
    const z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * Math.PI * 3000.0 / 180.0);
    const theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * Math.PI * 3000.0 / 180.0);
    const bdLng = z * Math.cos(theta) + 0.0065;
    const bdLat = z * Math.sin(theta) + 0.006;

    return [bdLat, bdLng];
}

function transformLat(x, y) {
    return -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y +
           0.1 * x * y + 0.2 * Math.sqrt(Math.abs(x)) +
           ((20.0 * Math.sin(6.0 * x * Math.PI) +
             20.0 * Math.sin(2.0 * x * Math.PI) +
             20.0 * Math.sin(y * Math.PI) +
             40.0 * Math.sin(y / 3.0 * Math.PI) +
             160.0 * Math.sin(y / 12.0 * Math.PI) +
             320 * Math.sin(y * Math.PI / 30.0)) * 2.0 / 3.0);
}

function transformLng(x, y) {
    return 300.0 + x + 2.0 * y + 0.1 * x * x +
           0.1 * x * y + 0.1 * Math.sqrt(Math.abs(x)) +
           ((20.0 * Math.sin(6.0 * x * Math.PI) +
             20.0 * Math.sin(2.0 * x * Math.PI) +
             20.0 * Math.sin(x * Math.PI) +
             40.0 * Math.sin(x / 3.0 * Math.PI) +
             150.0 * Math.sin(x / 12.0 * Math.PI) +
             300.0 * Math.sin(x / 30.0 * Math.PI)) * 2.0 / 3.0);
}
```

### **二、所需数据**

#### **1. 停车场数据（经纬度使用WGS84坐标）**

```javascript
const parkingLots = [
    { name: "凤凰居1号宿舍楼下", lat: 36.3593, lng: 120.6846, address: "凤凰居1号楼北侧指定停车区", polygon: [
        {lat: 36.3591, lng: 120.6844},
        {lat: 36.3591, lng: 120.6848},
        {lat: 36.3595, lng: 120.6848},
        {lat: 36.3595, lng: 120.6844},
        {lat: 36.3591, lng: 120.6844}]},
    { name: "教学楼", lat: 36.3627, lng: 120.6823, address: "教学楼东楼", polygon: [
        {lat: 36.3625, lng: 120.6821},
        {lat: 36.3625, lng: 120.6825},
        {lat: 36.3629, lng: 120.6825},
        {lat: 36.3629, lng: 120.6821},
        {lat: 36.3625, lng: 120.6821}]},
    { name: "食堂西门", lat: 36.3598, lng: 120.6878, address: "食堂北门出口处", polygon: [
        {lat: 36.3596, lng: 120.6876},
        {lat: 36.3596, lng: 120.6880},
        {lat: 36.3600, lng: 120.6880},
        {lat: 36.3600, lng: 120.6876},
        {lat: 36.3596, lng: 120.6876}]},
    { name: "图书馆南门", lat: 36.3659, lng: 120.6836, address: "图书馆南门入口西侧", polygon: [
        {lat: 36.3657, lng: 120.6834},
        {lat: 36.3657, lng: 120.6838},
        {lat: 36.3661, lng: 120.6838},
        {lat: 36.3661, lng: 120.6834},
        {lat: 36.3657, lng: 120.6834}]},
    { name: "校医院北侧", lat: 36.3587, lng: 120.6851, address: "校医院西侧非机动车停放区", polygon: [
        {lat: 36.3585, lng: 120.6849},
        {lat: 36.3585, lng: 120.6853},
        {lat: 36.3589, lng: 120.6853},
        {lat: 36.3589, lng: 120.6849},
        {lat: 36.3585, lng: 120.6849}]},
    { name: "体育馆东门", lat: 36.3589, lng: 120.6813, address: "体育馆东门入口南侧", polygon: [
        {lat: 36.3587, lng: 120.6811},
        {lat: 36.3587, lng: 120.6815},
        {lat: 36.3591, lng: 120.6815},
        {lat: 36.3591, lng: 120.6811},
        {lat: 36.3587, lng: 120.6811}]}
];
```

#### **2. 随机生成车辆**

考虑命名车辆为（A ~ F）➕ xxx，规定超出中心点10m的电动自行车不算入停车点。采用经纬度偏移的方法在每个停车点周围生成车辆，同时随机生成每辆电动自行车的信息，包括电量，ID，状态等。生成300辆电动自行车，随机分布在随机停车点周围。

```javascript
function generateVehicles() {
    const vehicles = [];
    const prefixes = ['A', 'B', 'C', 'D', 'E', 'F'];
    for(let i = 0; i < 300; ++i)
    {
        const baseLot = parkingLots[Math.floor(Math.random() * parkingLots.length)]; // 每次随机选取一个停车点，floor()向下取整和random()生成[0,1)的数，使选取每个停车点的机会相等
        const offsetLat = (Math.random() - 0.5) * 0.0004;//设置纬度偏移量，范围[-0.0002, 0.0002);
        const offsetLng = (Math.random() - 0.5) * 0.0004;//设置经度偏移量，范围[-0.0002, 0,0002);
        vehicles.push({
            id: `${prefixes[Math.floor(Math.random() * prefixes.length)]}${String(i + 1).padStart(3, '0')}`,// 随机等机会取字母编号，设置1 - 300的编号，并将编号转换为字符串，padStart让长度不足3的字符串的用字符‘0’补齐前位
            lat: baseLot.lat + offsetLat, // offsetLat的取值范围保证了车辆能在停车点中心点的南北向随机分布
            lng: baseLot.lng + offsetLng, // offsetLng的取值范围保证了车辆在停车点中心点的东西向随机分布
            battery: Math.ceil(Math.random() * 100) + '%',//使车辆电量在0 - 100%随机分布，模拟现实情况
            status: Math.random() > 0.1 ? 'available' : 'in_use'// 使空闲和使用中的车辆比例为9:1。
        });
    }
    return vehicles;
}

const vehicles = generateVehicles();
```

#### **3. 求地球上两点间的距离**

这里使用Haversine公式计算地球上两点间的距离。注意这里需要使用WGS84坐标下的经纬度。

**Haversine公式：**
$$
d = 2R \arcsin\left(\sqrt{\sin^2\left(\frac{\phi_2-\phi_1}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\lambda_2-\lambda_1}{2}\right)}\right)
$$

这里$R$是地球半径，$\phi_1, \phi_2$是纬度，$\lambda_1, \lambda_2$是经度。

```javascript
function calculateDistance(lat1, lat2, lng1, lng2) {
    const R = 6371000; //地球半径
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLng = (lng2 - lng1) * Math.PI / 180; //转换为弧度制
    const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) + Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) * Math.sin(dLng / 2) * Math.sin(dLng / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return R * c;
}
```

### **三、与前端交互**

所需工具中的变量app接收了express应用对象，挂载了很多方法，下面用这些方法来与前端实现交互。

#### **1. 获取停车场详情**

使用get请求路由，syntax：`app.get(path, handler)`. path是URL路径，一般用`'/api/...'`来表示这是提供数据的接口；handler处理函数 `(req, res) => { … }`，请求到达时执行，其中req是前端发来的请求，res是服务器返回前端的对象。

```javascript
app.get('/api/parking-lots', (req, res) => {
    const lotsWithCount = parkingLots.map(lot => { // map遍历数组并返回一个新数组
        const vehiclesInLot = vehicles.filter(v => { // filter遍历数组筛选符合条件的值并返回一个新数组
            const [lat1, lng1] = [lot.lat, lot.lng];
            const [lat2, lng2] = [v.lat, v.lng];
            const distance = calculateDistance(lat1, lat2, lng1, lng2);
            return distance <= 10 && v.status === 'available' && v.battery !== '0%';
        });
        const [bdLat, bdLng] = wgs84ToBd09(lot.lat, lot.lng);
        const bdPolygon = lot.polygon.map(p => {
            const [bdLat, bdLng] = wgs84ToBd09(p.lat, p.lng);
            return { lat: bdLat, lng: bdLng };
        });
        return {
            name: lot.name,
            lat: bdLat,
            lng: bdLng,
            address: lot.address,
            vehicleCount: vehiclesInLot.length,
            polygon: bdPolygon
        };
    });
    res.json(lotsWithCount); // 以json的方式将数据返回前端
});
```

#### **2. 获取指定停车场内的车辆详情**

这里同样使用get请求路由。

```javascript
app.get('/api/parking-lots/:name/vehicles', (req, res) => { // 这里面的:name是动态参数, :是动态占位符，由于这里要求获取指定停车场详情，故这里的name应该动态指定。
    const lotName = decodeURIComponent(req.params.name); // 前端发来的req的动态参数存放在param里，由于URL里会出现中文或特殊符号，因此我们需要在前端将该动态参数用encodeURIComponent()编码，然后在这里解码，以保证该参数的安全传输，因为浏览器自动编码可能会出错
    const lot = parkingLots.find(l => l.name === lotName); // 找到parkingLots中和动态参数名字一致的停车场。
    if(!lot){
        return res.status(404).json({ error: '停车点不存在'}); // 如果不存在，设置HTTPS的状态码为404，表示请求资源不存在，并将错误以json格式返回前端。
    }
    const vehiclesInlot = vehicles.filter(v => {
        const [lat1, lng1] = [lot.lat, lot.lng];
        const [lat2, lng2] = [v.lat, v.lng];
        const distance = calculateDistance(lat1, lat2, lng1, lng2);
        return distance <= 10 && v.status === 'available' && v.battery !== '0%';
    }).map(v => {
        const [lat1, lng1] = [lot.lat, lot.lng];
        const [lat2, lng2] = [v.lat, v.lng];
        const distance = calculateDistance(lat1, lat2, lng1, lng2);
        const [bdLat, bdLng] = wgs84ToBd09(lat2, lng2);
        return {
            id: v.id,
            battery: v.battery,
            distance: Math.round(distance) + 'm',
            lat: bdLat,
            lng: bdLng
        };
    }); // vehicles.filter().map() 意思是先筛选出一个符合标准的数组，再返回一个对象数组。JS从左到右执行。
    res.json(vehiclesInLot);
});
```

#### **3. 根据用户位置筛选出最近的停车场**

由于这个功能需要前端返回用户位置，因此使用POST路由。POST路由可以传递一个请求体body，也就是可以将用户经纬度包括在body里面；并且用户经纬度属于敏感信息，用GET从前端传递到后端会将用户经纬度暴露在URL上，例如：`` `fetch('/api/nearby-parking-lots?lat=31.2&lng=121.5')` ``

```javascript
app.post('/api/nearby-parking-lots', (req, res) => {
    const {lat, lng} = req.body;

    if(!lat || !lng){
        return res.status(400).json({ error: '缺少位置信息'});//400状态码表示前端发来的数据有问题，404表示请求的资源不存在 
    }

    const lotsWithDistance = parkingLots.map(lot => {
        const [lat1, lng1] = [lat, lng];
        const [lat2, lng2] = [lot.lat, lot.lng];
        const distance = calculateDistance(lat1, lat2, lng1, lng2);
        const vehicleInLot = vehicles.filter(v => {
            const [lat1, lng1] = [lot.lat, lot.lng];
            const [lat2, lng2] = [v.lat, v.lng];
            const distance = calculateDistance(lat1, lat2, lng1, lng2);
            return distance <= 10 && v.status === 'available' && v.battery !== '0%';
        });
        const [bdLat, bdLng] = wgs84ToBd09(lat2, lng2);
        return {
            name: lot.name,
            lat: bdLat,
            lng: bdLng,
            address: lot.address,
            distance: Math.round(distance),
            vehicleCount: vehicleInLot.length
        };
    });

    lotsWithDistance.sort((a, b) => a.distance - b.distance); // 排序，近的在前，远的在后
    res.json(lotsWithDistance);
});
```

#### **4. 启动服务器，监听指定端口**

app.listen(PORT), 启动HTTP服务器，监听指定端口号PORT，当有客户端的请求时，Express开始处理，并通过回调函数在控制台打印服务器运行信息。

```javascript
app.listen(PORT, () => {
    console.log(`服务器运行在 http://localhost:${PORT}`);
});
```

如此我们完成了后端设计，它能为我们提供：

- 每个停车场的名字，中心点经纬度，详细地址，车辆数；

- 指定停车场里面的车辆详情，包括id，电量，距停车点中心点距离，车辆的经纬度；

- 按与用户的距离从小到大顺序排列好的停车场及其详细信息，包括停车场的名字，中心点经纬度，详细地址，车辆数，与用户的距离。

并且能够监听端口，响应客户端的请求。

## **前端设计**

前端中我们需要用html来绘制页面结构，css来美化页面，js来添加功能。还要接入百度地图功能，已实现在地图上查看停车点和导航到停车点

### **一、HTML页面设计**

```html
<!DOCTYPE html>
<html lang="zh-CN"> <!--标注页面主要语言是中文-->

<head>
    <meta charset="UTF-8"> <!--使用UTF-8标准编码-->
    <meta name="viewport" content ="width=device-width, initial-scale=1.0"> <!--声明按照实际设备宽度视口（浏览器窗口）渲染网页，阻止默认缩小，并设置初始缩放比例为1.0，防止在一些浏览器上进行不确定的缩放。同时content ="width=device-width"还确保@media响应式设计能够触发-->
    <title>地图找车</title>
    <link rel="stylesheet" href="sytle.css"> <!--链接css-->
    <script type="text/javascript" src="https://api.map.baidu.com/api?v=3.0&ak=pmEiHxWExMjQk9E3VlaYOy4Zl11N8fF5"></script><!--指定脚本类型为javascript，加载百度地图官方提供的服务接口-->
</head>

<body>
    <div class="backgroundContainer"><!--设置背景容器，用于承载背景图片，地图入口-->
        <button class="header" id="showMap"> <!--class定义该元素类别，便于批量进行css渲染，id则用来让js精确选到该元素-->
            我心我行澄如明镜（点击有惊喜）
        </button>
        <img src="../../res/images/Backgroud.PNG" class="backgroundImage"> <!--从项目组中导图片-->
    </div> 

    <div class="mapContainer" style="display: none;" id="mapContainer"><!--header按钮呼出地图，于是需要设置地图容器来承载地图。由于反复加载地图太慢，于是默认将其隐藏，使其能够实时加载，后续只用使用js取消隐藏即可-->
        <div class="mapHeader">
            <button class="closeMapBtn" id="closeMapBtn">x</button>
            <h3>停车点地图导航</h3><!--将map header设置为第三级标题，大小合适-->
        </div>
        <div class="baiduMap" id="baiduMap"></div><!--在js接入百度地图并显示在这-->
    </div>

    <div class="contentContainer" id="parkingContainer"><!--用于承载主页面元素 将在后续用js从后端获取数据后添加-->
        <div class="loading">加载中...</div><!--防止主页面未加载出来被用户认为卡住-->
    </div>

    <div class="buttonPosition">
        <button class="filterBtn" id="filterBtn"><!--添加筛选按钮，后续功能在js中添加-->
            寻找附近
        </button>
    </div>

    <script src="function.js"></script> <!--添加js路径-->
</body>
</html>
```

### **二、CSS页面渲染**

用css来实现页面渲染。

```css
body {
    font-family: Arial, Helvetica, sans-serif;
    background-color: white;
    margin: 0;
    padding: 0;
    position: relative;
}
/*
首先设置网页默认字体，网页背景为白色。由于大多数浏览器会对body元素添加默认边距，于是将内外边距设置为0，确保与浏览器的绝对边缘对齐，便于控制。这里将position设置为relative，主要是给后续body上position为absolute的元素提供定位参考基准，也就是可以用绝对定位来精确控制元素在body上的位置。
*/

.backgroundContainer {
    position: fixed;/*由于我想让页面不随滚动条滚动而滚动，因此设置为fixed，这样该容器会相对于浏览器viewport进行定位，不会滚动*/
    top: 0;/*将容器与上边界对齐*/
    max-width: 600px;/*设置该容器最大宽度*/
    width: 100%;/*使容器宽度变为最大宽度的100%*/
    left: 50%;
    transform: translateX(-50%); /*现将容器左边缘放置在viewport宽度的50%处，即将左边缘居中，再让元素沿着X轴向左平移自身宽度的一半，最终达到元素居中效果*/
    overflow: hidden; /*溢出隐藏，即若容器内元素（背景图片）超过容器边界，则裁剪隐藏*/
    z-index: -1;/*将背景容器层叠顺序设为-1，防止覆盖容器内元素*/
}

.backgroundImage {
    width: 100%; /*占满容器*/
    height: auto; /*根据宽度和图片原始宽高比自动调整高度，防止图片拉伸变形*/
    object-fit: cover; /*确保图片能够完全覆盖容器，若宽高比与容器不匹配，则确保图片被裁剪而非拉伸*/
}

.header {
    display: flex; /*启用Flexbox弹性布局，使子元素灵活排列*/
    flex-direction: column; /*设置Flex项目从上到下竖直排列，此时主轴为y，交叉轴为x*/
    align-items: center; /*将Flex项目沿交叉轴水平对齐*/
    font-size: 1.2rem;/*根元素<html>的字体大小有默认值，1.2rem就是默认值的1.2倍。有几个好处：在进行响应式设计的时候只需要在Media Query里面更改<html>默认字体大小就可以完成缩放操作, 并且当用户更改浏览器字体默认大小时，其也能按比例自动缩放*/
    font-weight: bold;/*设置粗体*/
    background-color: rgba(198, 26, 26, 0.8);/*rgba中的a是透明度*/
    color: rgba(255, 255, 255, 1);
    padding: 15px 10px; /*简写为两个，为内容上下留出15px垂直空间，左右留出10px水平空间。 注意：若简写为4个，则按照上右下左的顺时针排列进行控制*/
    position: fixed;
    max-width: 600px;
    width: 100%;
    z-index: 1000; /*防止被覆盖*/
    border: none; /*无边框形式*/
    cursor: pointer; /*当鼠标悬停在该元素上，鼠标变为pointer*/
    transition: all 0.3s ease; /*添加过渡效果，当元素的任何属性发生变化时，动画将在0.3s内平滑过渡*/
}
.header:hover {/*添加鼠标悬停伪类，即当鼠标悬停在该元素上时，添加属性变化，比如放大等，提示用户鼠标所在位置*/
    background-color: rgba(198, 26, 26, 0.95);
    transform: scale(1.02);/*使用2D变换，让整个元素放大2%*/
}
.header:active {/*添加鼠标点击伪类，即当鼠标点击该元素时，添加属性变化*/
    transform: scale(0.98);/*模拟按压效果*/
}

.mapContainer { 
    display: flex; 
    flex-direction: column;
    position: fixed;
    top: 0;
    left: 0;/*从左上角出发*/
    width: 100%;
    height: 100%; /*占满整个viewport*/
    background: white;
    z-index: 3000; /*保证地图容器处于最上层*/
}

.mapHeader { /*Header容器 位于地图容器中 紧贴地图容器的最顶部自上而下排列*/
    display: flex; /*默认是水平排列*/
    background-color: rgba(198, 26, 26, 0.95);
    color: white;
    padding: 15px;
    align-items: center; /*在交叉轴（y轴）上居中，注意这里由于地图容器给他分配了一个空间，因此居中是在该空间里居中*/
    justify-content: space-between; /*第一个元素紧贴主轴最左端，最后一个元素紧贴主轴最右端 其他元素在这两个元素直接均匀分布*/
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}
.mapHeader h3 {
    margin: 0;
    font-size: 1.2rem;
    flex-grow: 1;
    text-align: center;
}

.closeMapBtn {
    background: rgba(255, 255, 255, 0.2);
    border: 2px solid white;
    color: white;
    padding: 8px 15px;
    border-radius: 5px;
    cursor: pointer;
    font-weight: bold;
    font-size: 1rem;
    transition: all 0.3s ease;
}
.closeMapBtn:hover {
    background: rgba(255, 255, 255, 0.3);
}

.baiduMap {
    flex: 1; /*占据父元素剩余的所有空间*/
    width: 100%;
    height: 100%;
}

.contentContainer { /*设置主界面容器*/
    display: flex;
    flex-direction: column; /*让主界面元素在竖直方向上排列*/
    max-width: 600px;
    margin: 120px auto 100px auto; /*左右auto使容器在body中水平居中，上120px和下100px，为header和筛选按钮留出空间*/
    gap: 50px; /*每个元素之间空50px*/
    padding: 0 1rem;/*主要设置左右内边距，确保其在小屏边缘不会紧贴设备边缘*/
}

.loading { /*设置“加载中...”元素样式*/
    text-align: center; /*设置元素内部文本内容水平居中*/
    padding: 40px; /*为文本的四个方向都留出40px的空间*/
    font-size: 1.1rem; /*比正文稍大，有提示作用*/
    color: white;
}

.buttonPosition {
    position: fixed;
    bottom: 20px;/*将元素下边缘固定在距离底部20px的位置上*/
    left: 50%;
    transform: translateX(-50%);
    z-index: 1000;
}

.filterBtn {
    padding: 15px 30px;
    font-size: 1rem;
    font-weight: bold;
    background-color: rgba(255, 255, 255, 0.9);
    color: rgba(2, 95, 255, 0.9);
    border: 2px solid rgba(2, 95, 255, 0.8); /*设置边框宽度，样式为solid实线边框，颜色*/
    border-radius: 40px; /*设置圆角半径，使按钮呈现药丸状*/
    cursor: pointer;
    transition: all 0.3s ease;
}
.filterBtn:hover {
    background-color: rgba(2, 95, 255, 0.9);
    color: white;
    transform: scale(1.05);
}
```

如此完成了当前页面下的css渲染设计，在js中还会添加一些元素，届时再利用css进行渲染

### **三、JavaScript功能添加**

Javascript前端设计需要从后端获取数据，完成主页面的设计以及交互功能。并且通过百度地图官方api接口导入百度地图。

#### **1. 准备工作**

```javascript
const API_BASE_URL = 'http://localhost:3000/api'; // 定义后端接口的基础地址，因为我的Express服务器监听localhost:3000,所有的后端接口都是/api/...开头
let userLocation = null; // 承载用户位置BD09版本，获取前定义为null
let userLocationwgs = null; // 承载用户位置的wgs84版本
let baiduMap = null; // 承载百度地图页面
let parkingLotsData = []; // 承载从后端获取的停车场数据对象

document.addEventListener('DOMContentLoaded', async () => { // 添加事件监听器，当'DOMContentLoaded‘即DOM树加载完毕时，调用这个异步回调函数。要注意这里的async是声明内部可以使用await，因此它本身并未直接发起异步请求。
    await initUserLocation(); // await作用是暂停当前函数（回调函数）的后续操作,等待异步操作完成。await操作本身在主线程中执行，不会阻塞主线程。此时initUserLocation()会继续执行并会返回一个Promise，而await下面的代码会被事件循环放入微任务队列中。直到主线程空闲且Promise resolve或reject，被暂停的函数才会重新进入主线程，继续执行。
    await loadParkingLots(); // 这里加入异步操作主要目的是暂停该回调函数，等loadParkingLots()获取到停车点数据后再返回，防止地图和筛选按钮提前加载。
    setupFilterButton(); // 加载筛选按钮
    setupMapButton(); // 加载地图按钮
});
```

#### **2. 初始化用户位置**

```javascript
async function initUserLocation(){
    return new Promise((resolve) => { // 创建一个要返回的Promise，用来标明resolve的情况
        if(!navigator.geolocation){ // 通过浏览器API获得用户经纬度
            showError('定位失败'); // 错误函数，最后会定义
            resolve(); // resolve 不传参数，标明解决但没有结果。
            return; // 提前退出函数
        }
        navigator.geolocation.getCurrentPosition( // getCurrentPosition是浏览器异步获取用户当前位置的方法，需要两个回调函数，用于获取成功触发和获取失败触发
            (position) => {
                const wgsLng = position.coords.longitude; // 获取WGS84坐标系下的经度
                const wgsLat = position.coords.latitude; // 获取WGS84坐标下的纬度
                userLocationwgs = {lat: wgsLat, lng: wgsLng};
                const convertor = new BMap.Convertor(); // 百度地图提供的WGS84 -> BD09的转换器
                const pointArr = [new BMap.Point(wgsLng, wgsLat)];//建立需要转换的目标点组，里面是百度地图坐标对象（由于translate的参数限制，即使只有一个点也需要用数组)
                convertor.translate(pointArr, 1, 5, (data) => {//回调函数
                    if(data.status === 0){  // 若status = 0 则转换成功
                        userLocation = {
                            lat: data.points[0].lat,
                            lng: data.points[0].lng
                        }; // 传入userLocation对象
                        resolve(); // 需要注意的是，在该函数中，主要目的是向userLocation中传入对象，因此只需要返回resolve表明解决，不需要返回任何结果
                    }
                    else{ // 转换失败
                        showError('定位失败');
                        resolve();
                    }
                });
            },
            (error) => { // 获取坐标失败触发
                console.error('定位失败', error); // 打印错误信息到控制台，括号里的error是getCurrentPosition回调返回的错误对象，里面有错误码和信息
                showError('无法获取位置信息');
                resolve();
            },
            {
                enableHighAccuracy: true, // 尽量使用GPS精确定位
                timeout: 10000, // 设置10s超时
                maximumAge: 0 // 不使用缓存，强制使用最新位置
            }
        );
    });
}
```

#### **3. 加载停车场列表**

```javascript
async function loadParkingLots() { 
    try { //尝试可能出错的异步操作，若出错则catch error
        const response = await fetch(`${API_BASE_URL}/parking-lots`); // fetch异步向服务器发出HTTP请求，await等待请求完成
        parkingLotsData = await response.json(); // .json()异步解析返回的json数据为普通js对象，await等待请求完成
        renderParkingLots(parkingLotsData); // 渲染停车场卡片，即主页面元素
    }
    catch(error) {
        console.error('', error);
        showError('加载数据失败，请检查后端服务是否运行')
    }
}
// 这里隐式返回Promise，若没有catch到error，则返回resolve，由于没有写return，故resolve里的值是undefine；若catch到error，则返回reject
```

#### **4. 添加并渲染停车场卡片，即主页面元素**

```javascript
function renderParkingLots(parkingLots)
{
    const container = document.getElementById('parkingContainer'); // 获取id为parkingContainer的元素
    container.innerHTML = ''; // 将container中的“加载中...”移除
    
    parkingLots.forEach(lot => { // 遍历所有停车点
        const card = document.createElement('div'); // 创造一个div元素
        card.className = 'parkingLot'; // 该元素的类名
        card.innerHTML = `
            <h3>${lot.name}</h3>
            <p>详细信息：${lot.address}</p>
            <p>中心点：${lot.lat}, ${lot.lng}</p>
            <p><strong>可用车辆：${lot.vehicleCount} 辆</strong></p>
            ${lot.distance ? `<p class="distance">距离您：${lot.distance}米</p>` : ''}
        `; // div内部的html内容。注意这里的``反引号用来进行${}插值操作
        card.addEventListener('click', () => {
            showParkingInfo(lot.name);
        }); // 添加点击交互内容
        container.appendChild(card); // 将card加入到container中，完成主页面绘制
    });
}
```

这里加入到html中的元素依旧需要css渲染：

```css
.parkingLot {
    background-color: rgba(255, 255, 255, 0.85);
    padding: 30px 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2); /*添加阴影效果，第一个值是水平偏移量，第二个值是垂直偏移量（向下移动2px），第三个值是模糊半径，即模糊程度*/
    cursor: pointer;
    transition: all 0.3s ease;
}
.parkingLot:hover {
    transform: translateY(-5px); /*沿y轴向上移动5px，默认y轴方向垂直向下*/
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}
.parkingLot h3 { /*渲染<h3>元素*/
    margin: 0 0 10px 0;
    color: rgb(198, 26, 26);
    font-size: 1.3rem;
}
.parkingLot p { /*渲染<p>元素*/
    margin: 8px 0;
    color: rgb(51, 51, 51);
}
.parkingLot .distance{ /*渲染parkingLot下面的distancce类*/
    color: rgb(2, 95, 255);
    font-weight: bold;
    font-size: 1.1rem;
}
```

#### **5. 点击卡片显示停车点车辆详情**

```javascript
async function showParkingInfo(name) {
    try {
        const response = await fetch(`${API_BASE_URL}/parking-lots/${encodeURIComponent(name)}/vehicles`); // 编码name，防止出现乱码导致传递失败
        const vehicles = await response.json();
        
        if(vehicles.length === 0) // 没有车辆
        {
            showMessage('该停车场暂无可用车辆') // 新函数，用来处理特殊情况
            return;
        }
        const lot = parkingLotsData.find(l => l.name === name); // 找到里面同名的停车点数据
        const popup = document.createElement('div');
        popup.className = 'popupOverlay';
        popup.innerHTML = `
            <div class="popup">
                <h3>${name} 车辆信息</h3>
                <p class="vehicleCount">共 ${vehicles.length} 辆可用</p>
                <ul>
                    ${vehicles.map(v => `
                    <li>
                        <strong>编号：</strong>${v.id}
                        <strong>电量：</strong>${v.battery}
                        <strong>距中心：</strong>${v.distance}
                    </li>`).join('')}
                </ul>
                <button class="navigationButton" onclick="navigateToParking('${lot.name}', ${lot.lat}, ${lot.lng})">
                    导航
                </button>
                <button class="closeButton">关闭</button>
            </div>
        `;
        // <ul>的意思是unordered list，默认是黑点列表；.map()的作用是遍历数组，进行回调函数的操作后返回一个新数组；.join('')的作用是将数组中的内容链接起来。这样就是<ul>下含有多个<li>，显示效果则为按照list的方式排列。
        document.body.appendChild(popup);

        popup.addEventListener('click', e => {
            if(e.target.classList.contains('popupOverlay') || e.target.classList.contains('closeButton')){
                popup.remove();
            } // 这里目的是 点击关闭按钮和弹窗外的部分都能移除弹窗 target就是点击的位置
        });
    }
    catch(error){
        console.error('加载车辆信息失败', error);
        showError('加载车辆信息失败');
    }
}
```

这里加入到html中的元素依旧需要css渲染：

```css
.popupOverlay { /*这里要模拟弹出详细信息时页面变黑的操作*/
    display: flex;
    flex-direction: column;
    position: fixed;
    top: 0;
    left: 0; /*从左上角开始延伸*/
    width: 100%;
    height: 100%; /*占满整个body*/
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.5);
    z-index: 2000;
    animation: fadeIn 0.3s ease; /*添加淡入的动画效果，0.3s内平滑过渡*/
}
@keyframes fadeIn { /*添加keyframe规则，设置淡入的动画效果*/
    from {
        opacity: 0;
    }
    to {
        opacity: 1; 
    }
}
/*不透明度从0 - 1，1也就是background原本的颜色rgba(0, 0, 0, 0.5)*/

.popup {
    background: white;
    padding: 30px 40px;
    border-radius: 20px;
    max-width: 500px;
    width: 90%;
    max-height: 80vh; /*最大高度是viewport高度的80%，防止内容过多超出屏幕*/
    overflow-y: auto; /*如果内容过多溢出，则会启动滚动条，保证所有内容均可见*/
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    animation: slideUp 0.3s ease; /*添加向上滑动动画 模拟卡片从下到上 弹窗效果*/
}
@keyframes slideUp {
    from {
        transform: translateY(50px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}
.popup h3 {
    margin-top: 0;
    margin-bottom: 15px;
    color: rgb(198, 26, 26);
    font-size: 1.4rem;
}
.popup .vehicleCount {
    color: rgb(2, 95, 255);
    font-weight: bold;
    margin-bottom: 15px;
}
.popup ul {
    list-style: none; /*去除默认黑点*/
    padding: 0;
    margin: 0;
}
.popup li {
    margin-bottom: 15px;
    padding: 12px;
    background: rgb(245, 245, 245);
    border-radius: 8px;
    border-left: 4px solid rgb(2, 95, 255);/*仅在左边缘添加颜色*/
}
.popup li strong {
    color: rgb(51, 51, 51);
    margin-right: 8px;
}

.navigationButton {
    display: block;
    margin: 10px auto 0 auto;
    padding: 12px 30px;
    border: 2px solid rgba(2, 95, 255, 0.9);
    background: white;
    color: rgba(2, 95, 255, 0.9);
    border-radius: 25px;
    cursor: pointer;
    font-size: 1rem;
    font-weight: bold;
    transition: all 0.3s ease;
}
.navigationButton:hover {
    background: rgba(2, 95, 255, 0.9);
    color: white;
    transform: scale(1.05);
}

.closeButton {
    display: block; /*块级元素 独占一行*/
    margin: 20px auto 0 auto; /*依旧左右对齐*/
    padding: 12px 30px;
    border: none;
    background: rgba(2, 95, 255, 0.9);
    color: white;
    border-radius: 25px;
    cursor: pointer;
    font-size: 1rem;
    font-weight: bold;
    transition: all 0.3s ease;
}
.closeButton:hover {
    background: rgba(2, 95, 255, 1);
    transform: scale(1.05);
}
```

#### **6. 设置寻找附近按钮**

```javascript
function setupFilterButton() {
    const filterBtn = document.getElementById('filterBtn');
    filterBtn.addEventListener('click', async () => {
        if(!userLocation){
            showError('无法定位');
            return;
        }
        filterBtn.textContent = '定位中...';// 点击之后开始定位
        filterBtn.disabled = true; // 这里是为了防止用户重复点击导致重复定位，disabled = true就是不可点击。因此在css样式设计上，filterBtn还需要更改
        try{
            const response = await fetch(`${API_BASE_URL}/nearby-parking-lots`, {
                method: 'POST', //表示这是一个POST请求
                headers: { 'Content-Type': 'application/json' },//告诉服务器，前端发送的是JSON格式的文本
                body: JSON.stringify(userLocationwgs) // 由于网络传输只能传字符串，因此用JSON.stringify()将用户地址转成JSON字符串.将wgs坐标传过去，防止计算错误
            }); // fetch()默认发出get请求，这里要将用户位置传到后端，故要使用POST请求
            const nearbyLots = await response.json();
            renderParkingLots(nearbyLots);//使用相同的渲染方式并更改主界面卡片顺序
            showNearbyPopup(nearbyLots); //新函数，展示附近停车点弹窗

            filterBtn.textContent = '重新寻找' // 更改按钮中的字符
            filterBtn.disabled = false; // 设置为false可以点击
        }
        catch(error) {
            console.error('寻找失败：', error);
            showError('寻找失败，请重试');
            filterBtn.textContent = '寻找附近';
            filterBtn.disabled = false;
        }
    });
}
```

更改filterBtn的css样式设计,仅更改伪类

```css
.filterBtn:hover:not(:disabled) { /*鼠标悬停在未被禁用的按钮上时，not是逻辑否定伪类，即不是某个状态的元素，:disabled意思是处于禁用状态的元素*/
    background-color: rgba(2, 95, 255, 0.9);
    color: white;
    transform: scale(1.05);
}
.filterBtn:disabled {
    opacity: 0.6; /*调低不透明度，使其看起来灰灰的*/
    cursor: not-allowed;/*鼠标悬停时光标变成 🚫 样式，表示禁止点击*/
}
```

#### **7.显示附近停车场弹窗**

```javascript
function showNearbyPopup(lots) {
    const popup = document.createElement('div');
    popup.className = 'popupOverlay';
    popup.innerHTML = `
        <div class="popup">
            <h3>距离最近的停车点</h3>
            <ul>
                ${lots.map((lot, index) => `
                <li>
                    <strong>${index + 1}. ${lot.name}</strong><br>
                    距离：${lot.distance}米 | 可用车辆：${lot.vehicleCount}辆
                </li>`).join('')}
            </ul>
            <button class="closeButton">关闭</button>
        </div>
    `; // <br>是换行符 index是当前元素的索引，从0开始 添加关闭按钮，随后在css里进行样式设计
    document.body.appendChild(popup);

    popup.addEventListener('click', e => {
        if(e.target.classList.contains('popupOverlay') || e.target.classList.contains('closeButton')) {
            popup.remove();
        }
    });
}
```

#### **8. 设置地图按钮（顶部模块）**

```javascript
function setupMapButton() {
    const showMapBtn = document.getElementById('showMap');
    const mapContainer = document.getElementById('mapContainer');
    const closeMapBtn = document.getElementById('closeMapBtn');
    
    showMapBtn.addEventListener('click', () => {
        showMap();
    });
    closeMapBtn.addEventListener('click', () => {
        mapContainer.style.display = 'none'; // 仅隐藏地图，若remove则要重新加载，性能消耗太大
    });
}
```

#### **9. 显示地图**

```javascript
function showMap() {
    const mapContainer = document.getElementById('mapContainer');
    mapContainer.style.display = 'flex';
    if(!userLocation) {
        showError('无法获取定位信息');
        return;
    }
    initBaiduMap(userLocation); //初始化百度地图，使之能够显示
    renderParkingLotsOnMap(); //渲染地图上的停车点 一方面要圈出区域 另一方面要用little dots 表示停车点内车辆具体位置
}
```

#### **10. 初始化百度地图**

```javascript
function initBaiduMap(centerLocation) { // 这里用centerLocation，是因为点开百度地图，百度地图需要固定一个中心点并进行缩放
    if(!baiduMap) { //若空，则要通过百度地图官方的BMap库进行初始化创建。BMap是百度地图Javascript SDK提供的全局命名空间对象，所有百度地图的功能都在里面
        baiduMap = new BMap.Map('baiduMap'); // 创建一个地图对象，.Map()接收一个id参数，并将该对象绑定在持有这个ID的<div>容器
        baiduMap.enableScrollWheelZoom(true); // 启用鼠标滚轮缩放功能
        baiduMap.addControl(new BMap.NavigationControl());// 添加导航控件
        baiduMap.addControl(new BMap.ScaleControl());// 添加比例尺控件
    }
    baiduMap.clearOverlays(); // 移除地图上的所有已有覆盖物，确保只存在下面定义的标记

    const point = new BMap.Point(centerLocation.lng, centerLocation.lat); //创建一个百度地图坐标点对象，这里用传入的中心点（用户位置）创建
    baiduMap.centerAndZoom(point, 16); // 将地图中心设为该点，缩放级别设置为16 这是一个能看清街道的级别

    setTimeout(() => {
        baiduMap.checkResize(); //重新计算大小
        baiduMap.centerAndZoom(point, 16); //重设中心点
    }, 200);// 延迟200ms再调用
    // 用来修复百度地图在动态显示容器时的“尺寸异常”和初始定位异常的问题
    
    const userMarker = new BMap.Marker(point); // 用用户经纬度创建一个标记点，这个标记点是百度地图默认样式
    userMarker.setTitle('您当前的位置'); // 给标记添加title属性，当鼠标悬停在标记上时title会出现，标明用户当前位置
    baiduMap.addOverlay(userMarker); // Map类的一个方法，作用是添加将这个标记点添加到地图上
    userMarker.setAnimation(BMAP_ANIMATION_BOUNCE); //设置标记上下跳动的动画
    setTimeout(() => userMarker.setAnimation(null), 1000); // 使动画持续1s
}
```

#### **11. 渲染地图上的停车场**

```javascript
async function renderParkingLotsOnMap() {
    if(!baiduMap || parkingLotsData.length === 0) return;

    baiduMap.clearOverlays();//渲染之前都把之前的覆盖物全部清除

    const userPoint = new BMap.Point(userLocation.lng, userLocation.lat);
    const userMarker = new BMap.Marker(userPoint);
    userMarker.setTitle('您当前的位置');
    baiduMap.addOverlay(userMarker);
    userMarker.setAnimation(BMAP_ANIMATION_BOUNCE);
    setTimeout(() => userMarker.setAnimation(null), 1000);
    
    const userInfoWindow = new BMap.InfoWindow("<strong>您当前的位置</strong>", {
        width: 150,
        height: 50
    }); //创建一个信息窗口，用来在地图上弹出提示框

    userMarker.addEventListener('click', () => {
        baiduMap.openInfoWindow(userInfoWindow, userPoint);
    });

    for(const lot of parkingLotsData) {
        const lotPoint = new BMap.Point(lot.lng, lot.lat);
        const lotMarker = new BMap.Marker(lotPoint);
        lotMarker.setTitle(lot.name);
        baiduMap.addOverlay(lotMarker);
        const lotInfo = `
            <div>
                <strong>${lot.name}</strong><br>
                可用车辆：${lot.vehicleCount} 辆<br>
                具体位置：${lot.address}
            </div>`;
        addInfoWindow(lotMarker, lotInfo);//新函数，添加停车点的详细信息窗口，并提供点击交互，跟用户位置的窗口和交互一致，封装成函数提高可读性

        //设置多边形停车区域
        const polygonPoints = lot.polygon.map(p => new BMap.Point(p.lng, p.lat)); //获取每个停车点coordinats数组
        const polygon = new BMap.Polygon(polygonPoints, {
            strokeColor: "blue", //设置多边形边界线蔚蓝色
            strokeWeight: 2, // 设置多边形边界宽度为2px
            strokeOpacity: 0.5, // 设置多边形边界不透明度
            fillColor: "rgb(160, 200, 255)", //设置多边形内部颜色
            fillOpacity: 0.3 //设置多边形内部不透明度
        });
        baiduMap.addOverlay(polygon);
        addInfoWindow(polygon, lotInfo);//点击停车场区域也会弹出详细信息

        const res = await fetch(`${API_BASE_URL}/parking-lots/${encodeURIComponent(lot.name)}/vehicles`);
        const vehicles = await res.json();
        vehicles.forEach(v => { //遍历车辆数据
            const vehiclePoint = new BMap.Point(v.lng, v.lat);//创建每辆车的坐标点
            const vehicleMarker = new BMap.Marker(vehiclePoint, { //设置自定义图标
                icon: new BMap.Icon( 
                    "https://api.map.baidu.com/images/marker_red_sprite.png", // 使用百度地图提供的红色精灵图
                    new BMap.Size(6, 6), // 图标实际显示尺寸为6x6px
                    { anchor: new BMap.Size(3, 3)} // 定义锚点为图标中心，即将中心与车辆坐标点重合。默认(0, 0)，即将图标左上角挂在车辆坐标点上
                )
            });
            baiduMap.addOverlay(vehicleMarker);

            const info = `<strong>车辆编号：</strong>${v.id}<br>电量：${v.battery}<br>距离停车点：${v.distance}`;
            addInfoWindow(vehicleMarker, info);
        });
    }
}
```

#### **12. 添加信息窗口**

```javascript
function addInforWindow(target, content) { // target是要添加信息窗口的对象，content是信息窗口内容
    const infoWindow = new BMap.InfoWindow(content, { //第二个参数设置信息窗口属性
        width: 200,
        height: 120,
        title: "详情",
        enableMessage: false //关闭信息窗口底部的“发送给手机”等短信发送功能按钮。
    });
    target.addEventListener('click', (e) => {
        baiduMap.openInfoWindow(infoWindow, e.point);
    });
}
```

#### **13. 导航到停车场**

```javascript
async function navigateToParking(name, lat, lng) {
    if(!userLocation) {
        showError('无法获取您的位置，请先允许定位权限');
        return;
    }

    const popup = document.querySelector('.popupOverlay');
    if(popup) popup.remove(); //在地图呈现之前先把弹窗去除

    const mapContainer = document.getElementById('mapContainer');
    mapContainer.style.display = 'flex'; // 显示地图

    if(!baiduMap){ // 如果地图还没初始化，就先初始化然后显示
        initBaiduMap(userLocation);
    }

    const start = new BMap.Point(userLocation.lng, userLocation.lat);
    const end = new BMap.Point(lng, lat); // 要去的停车点的经纬度
    baiduMap.clearOverlays(); // 照例清除所有标记

     const lot = parkingLotsData.find(l => l.name === name);
     if(lot)
     {
        const polygonPoints = lot.polygon.map(p => new BMap.Point(p.lng, p.lat));
        const polygon = new BMap.Polygon(polygonPoints, {
            strokeColor: "blue",
            strokeWeight: 2,
            strokeOpacity: 0.5,
            fillColor: "rgb(160, 200, 255)",
            fillOpacity: 0.3
        });
        baiduMap.addOverlay(polygon);
     }
     else{
        console.warn(`未获取到名为 "${name}" 的停车点数据。`);
     }

    const userMarker = new BMap.Marker(start);
    userMarker.setTitle('您当前的位置');
    baiduMap.addOverlay(userMarker);
    userMarker.setAnimation(BMAP_ANIMATION_BOUNCE);
    setTimeout(() => userMarker.setAnimation(null), 1000);

    const lotPoint = new BMap.Point(end.lng, end.lat);
    const lotMarker = new BMap.Marker(lotPoint);
    lotMarker.setTitle(lot.name);
    baiduMap.addOverlay(lotMarker);

    const res = await fetch(`${API_BASE_URL}/parking-lots/${encodeURIComponent(name)}/vehicles`);
    const vehicles = await res.json();
    vehicles.forEach(v => {
        const vehiclePoint = new BMap.Point(v.lng, v.lat);
        const vehicleMarker = new BMap.Marker(vehiclePoint, {
            icon: new BMap.Icon(
                "https://api.map.baidu.com/images/marker_red_sprite.png",
                new BMap.Size(6, 6),
                { anchor: new BMap.Size(3, 3) }
            )
        });
        baiduMap.addOverlay(vehicleMarker);

        const info = `<strong>车辆编号：</strong>${v.id}<br>电量：${v.battery}<br>距离停车点：${v.distance}`;
        addInfoWindow(vehicleMarker, info);
    });

    //添加导航路线
    const riding = new BMap.RidingRoute(baiduMap, { //创建一个骑行规划实例
        renderOptions:{map: baiduMap, autoViewPort: true},//autoViewPort: true: 自动调整视野
    });
    riding.search(start, end);
}
```

#### **14. 前面设置的信息显示**

- **显示错误信息**

```javascript
function showError(message) {
    const popup = document.createElement('div');
    popup.className = 'popupOverlay';
    popup.innerHTML = `
        <div class="popup">
            <h3>⚠️提示⚠️</h3>
            <p>${message}</p>
            <button class="closeButton">确定</button>
        </div>
    `;
    document.body.appendChild(popup);

    popup.addEventListener('click', e => {
        if (e.target.classList.contains('popupOverlay') || e.target.classList.contains('closeButton')) {
            popup.remove();
        }
    });
}
```

- **显示普通消息**

```javascript
function showMessage(message) {
    const popup = document.createElement('div');
    popup.className = 'popupOverlay';
    popup.innerHTML = `
        <div class="popup">
            <h3>💡提示💡</h3>
            <p>${message}</p>
            <button class="closeButton">确定</button>
        </div>
    `;
    document.body.appendChild(popup);

    popup.addEventListener('click', e => {
        if (e.target.classList.contains('popupOverlay') || e.target.classList.contains('closeButton')) {
            popup.remove();
        }
    }); 
}
```

### **四、剩余的响应式设计**

```css
@media (max-width: 480px) { /*@media是 Media Query媒体查询，只有当viewport宽度小于等于480px才会生效 480px主要对应智能手机的竖屏模式*/

/*下面的设计均为覆盖原先操作。若无原先操作，则添加*/
    .header {
        font-size: 1rem;
        padding: 12px;
    }

    .contentContainer {
        margin: 100px auto 80px auto;
        gap: 30px;
        padding: 0 0.5rem;
    }

    .parkingLot {
        padding: 20px 15px;
    }

    .parkingLot h3 {
        font-size: 1.1rem;
    }

    .filterBtn {
        padding: 12px 25px;
        font-size: 0.9rem;
    }

    .popup {
        padding: 20px 25px;
        width: 85%;
    }

    .popup h3 {
        font-size: 1.2rem;
    }

    .mapHeader h3 {
        font-size: 1rem;
    }
}
```

### **五、配置package.json文件**

```json
{
    "name": "parking-finder-backend", 
    "version": "1.0.0",
    "main": "server.js",
    "scripts": {
        "start": "node server.js",
        "dev": "nodemon server.js"
    },
    "keywords": ["parking", "location", "map"],
    "author": "Xiao Jinping",
    "license": "MIT", 
    "dependencies": { 
        "express": "^4.18.2"
    },
    "devDendencies": {
        "nodemon": "^3.0.1"
    }
}
```

整个项目的Github仓库：[Openlab_test2 项目仓库](https://github.com/3525613741/Openlab_test2)
