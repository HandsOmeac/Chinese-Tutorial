
绘制等值线地图，第二部分：添加交互
在第1部分中，您在Mapbox Studio样式编辑器中设置了美国人口密度数据的样式，并发布了一个新样式。在第2部分中，您将使用Mapbox GL JS使这种样式在交互中变得生动起来。
Mapbox Studio和Mapbox GL JS如何协同工作
在上一份指南中，您使用Mapbox Studio样式编辑器设计地图并创建样式。但是当你点击发布时，这个软件生成了什么？
什么是样式？
样式是绘制网页地图最重要的部分：它包含了在网页上绘制哪些特征以及如何绘制这些特征的所有规则。Mapbox Studio和Mapbox GL JS都与您的样式直接交互：Mapbox Studio样式编辑器是一个创建样式的可视化界面，Mapbox GL JS用于将样式添加到网页中，并通过添加和更改图层和源以响应浏览器事件来直接与之交互。
sources：链接到将在地图上设置样式的所有数据。使用Mapbox Studio样式编辑器创建样式时，源是您Mapbox帐户中的栅格和矢量切片集。
sprite：指向样式中使用的所有图片和图标的链接。
glyphs：指向样式中使用的所有字体的链接。
layers：源中数据应如何显示在地图上的规则列表。

当您在第1部分中将人口密度数据添加到Mapbox Studio样式时，指向该数据的链接也添加到样式对象的源列表中（我们通常将其称为“样式”）。同样地，当您添加人口密度图层并为其提供每个数据类别的样式规则时，该图层也会添加到样式中的图层列表中。

开始
以下是您需要开始的内容：
访问令牌。您可以在您的帐户页上找到访问令牌。
您的样式的样式URL。在样式页面中，点击人口密度样式旁边的菜单按钮，然后点击剪贴板图标复制样式URL。
Mapbox GL JS。用于构建网页地图的Mapbox JavaScript API。
文本编辑器。您毕竟要编写HTML,CSS, 和JavaScript。
创建网页
打开文本编辑器并创建一个名为index.html的文件。通过在头部添加Mapbox GL JS及其关联的CSS文件来配置文档：

<script src='https://api.mapbox.com/mapbox-gl-js/v1.3.1/mapbox-gl.js'></script>
<link href='https://api.mapbox.com/mapbox-gl-js/v1.3.1/mapbox-gl.css' rel='stylesheet' />

接下来，标记页面以创建地图容器、信息框和图例：
<div id='map'></div>
<div class='map-overlay' id='features'><h2>US population density</h2><div id='pd'><p>Hover over a state!</p></div></div>
<div class='map-overlay' id='legend'></div>

您还需要应用一些CSS来可视化布局的外观。这对于Map div尤其重要，除非您给它一个高度，否则它不会显示在页面上：
body {
  margin: 0;
  padding: 0;
}

h2,
h3 {
  margin: 10px;
  font-size: 1.2em;
}

h3 {
  font-size: 1em;
}

p {
  font-size: 0.85em;
  margin: 10px;
  text-align: left;
}

/**
* Create a position for the map
* on the page */
#map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}

/**
* Set rules for how the map overlays
* (information box and legend) will be displayed
* on the page. */
.map-overlay {
  position: absolute;
  bottom: 0;
  right: 0;
  background: rgba(255, 255, 255, 0.8);
  margin-right: 20px;
  font-family: Arial, sans-serif;
  overflow: auto;
  border-radius: 3px;
}

#features {
  top: 0;
  height: 100px;
  margin-top: 20px;
  width: 250px;
}

#legend {
  padding: 10px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  line-height: 18px;
  height: 150px;
  margin-bottom: 40px;
  width: 100px;
}

.legend-key {
  display: inline-block;
  border-radius: 20%;
  width: 10px;
  height: 10px;
  margin-right: 5px;
}

在下一步中，您将把地图添加到页面中，项目将开始成形。
初始化地图
现在您已经向页面添加了结构，就可以开始编写一些JavaScript 了！首先需要做的是添加访问令牌。没有这个，剩下的代码就不能生效。注意：以下所有代码都应该在脚本标记之间。
mapboxgl.accessToken = 'YOUR_MAPBOX_ACCESS_TOKEN';

现在您已经添加了页面的结构，您可以将一个Map对象添加到Map div中。请确保将样式URL替换为您在本指南第1部分中创建的样式的样式URL，否则代码不会生效！
var map = new mapboxgl.Map({
  container: 'map', // container id
  style: 'your-style-url' // replace this with your style URL
});

添加附加信息
在一些项目中，有些地方你需要停一下：你把地图放在一个页面上！但是对于这个地图，您将添加两个附加信息让地图更加实用：一个图例和一个信息窗体来显示光标悬停在任何地方的人口密度。

“加载”事件
什么是回调？
在页面上初始化地图不仅仅是在地图 div中创建一个容器，它还告诉浏览器请求你在第1部分中创建的mapbox studio样式。根据Mapbox服务器对该请求的响应速度，这可能需要花费不确定的时间，而代码中要添加的所有内容都依赖于加载到地图中的样式。因此，确保在执行更多代码之前加载样式非常重要。
幸运的是，地图对象可以告诉浏览器当地图的状态改变时发生的某些事件。其中一个事件是加载，当样式加载到地图上时会触发该事件。通过map.on方法，您可以将事件放在一个回调函数中（在加载事件发生时调用该函数），确保在该事件发生之前不会执行剩下的代码。

为了确保剩下的代码能够执行，它需要停留在一个回调函数中，该函数在地图加载完成时执行。
map.on('load', function() {
  // the rest of the code will go in here
});

创建图层间隔和颜色数组
创建包含州数据的图层样式时使用的点的列表允许我们在之后的步骤中向地图添加图例。
记住：这段代码位于加载回调函数的内部！
var layers = ['0-10', '10-20', '20-50', '50-100', '100-200', '200-500', '500-1000', '1000+'];
var colors = ['#FFEDA0', '#FED976', '#FEB24C', '#FD8D3C', '#FC4E2A', '#E31A1C', '#BD0026', '#800026'];

以下代码将向地图添加图例。为此，它将遍历您上面定义的图层列表，并根据层的名称及其颜色为每个层添加一个图例元素。
for (i = 0; i < layers.length; i++) {
  var layer = layers[i];
  var color = colors[i];
  var item = document.createElement('div');
  var key = document.createElement('span');
  key.className = 'legend-key';
  key.style.backgroundColor = color;

  var value = document.createElement('span');
  value.innerHTML = layer;
  item.appendChild(key);
  item.appendChild(value);
  legend.appendChild(item);
}

添加信息窗体
当光标悬停在某个州上时，信息窗口应显示该州的人口密度信息。如果光标没有悬停在某个状态上，则信息窗口应显示“悬停在某个州上！”
要执行此操作，请为mousemove事件添加一个监听器，确定光标所在位置的州（如果有的话），然后更新信息窗体：


map.on('mousemove', function(e) {
  var states = map.queryRenderedFeatures(e.point, {
    layers: ['statedata']
  });

  if (states.length > 0) {
    document.getElementById('pd').innerHTML = '<h3><strong>' + states[0].properties.name + '</strong></h3><p><strong><em>' + states[0].properties.density + '</strong> people per square mile</em></p>';
  } else {
    document.getElementById('pd').innerHTML = '<p>Hover over a state!</p>';
  }
});

最后润色
差不多了！最后几步：
光标
添加一行代码，为地图提供默认指针光标。
map.getCanvas().style.cursor = 'default';

地图边界
通过设置加载地图的边界，确保加载地图时显示美国大陆：
map.fitBounds([[-133.2421875, 16.972741], [-47.63671875, 52.696361]]);

任务完成
您已经创建了一个交互式等值线地图！
干得好！有关Mapbox Studio的更多操作，请参阅Mapbox Studio Manual。有关Mapbox GL JS及其工作方式的更多信息，请阅读How web apps work 。
