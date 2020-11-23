# 相关文档: https://developer.matomo.org/guides/tracking-javascript-guide

## JavaScript跟踪客户端
您可以使用JavaScript跟踪客户端来跟踪任何支持JavaScript的应用程序：例如网站！本指南将说明如何使用JavaScript跟踪客户端来自定义Matomo（以前称为Piwik）中记录某些网络分析数据的方式。

## 查找Matomo跟踪代码
要使用此页面中描述的所有功能，您需要使用最新版本的跟踪代码。要查找您网站的跟踪代码，请按照以下步骤操作：
- 使用您的管理员或超级用户帐户登录到Matomo
- 单击右上角菜单中的用户名，然后单击“设置”以访问管理区域
- 点击左侧菜单中的跟踪代码
- 在打开`<body>`标记之后（或在`<head>`部分中）将JavaScript跟踪代码复制并粘贴到您的页面中
跟踪代码如下所示：
```javascript
<!-- Matomo -->
<script type="text/javascript">
  var _paq = window._paq = window._paq || [];
  _paq.push(['trackPageView']);
  _paq.push(['enableLinkTracking']);
  (function() {
    var u="//{$MATOMO_URL}/";
    _paq.push(['setTrackerUrl', u+'matomo.php']);
    _paq.push(['setSiteId', {$IDSITE}]);
    var d=document, g=d.createElement('script'), s=d.getElementsByTagName('script')[0];
    g.type='text/javascript'; g.async=true; g.src=u+'matomo.js'; s.parentNode.insertBefore(g,s);
  })();
</script>
<!-- End Matomo Code -->
```
在您的跟踪代码中，`{$ MATOMO_URL}将替换为您的Matomo URL，{$ IDSITE}`将被替换为您正在Matomo中跟踪的网站的idsite。
对于熟悉JavaScript的人来说，这段代码可能看起来有些奇怪，但这是因为它是异步运行的。换句话说，浏览器将不会等待下载matomo.js文件来显示页面。
对于异步跟踪，独立于matomo.js的异步加载，将配置和跟踪调用推送到全局_paq数组上以执行。格式为：
```javascript
_paq.push([ 'API_method_name', parameter_list ]);
```
您还可以推送要执行的功能。例如：
```javascript
var visitor_id;
_paq.push([ function() { visitor_id = this.getVisitorId(); }]);
```
或者例如，使用异步代码获取自定义变量（名称，值）：
```javascript
_paq.push(['setCustomVariable','1','VisitorType','Member']);
_paq.push([ function() { var customVariable = this.getCustomVariable(1); }]);
```
即使已加载并运行matomo.js文件，也可以推送到_paq数组。
如果您的Matomo跟踪代码看起来不是这样，则您可能正在使用不建议使用的版本。较旧的版本仍然可以按预期运行，并且可以跟踪您的访问者，但是我们强烈建议您更新页面以使用最新的跟踪代码。

## 使用要求
### 支持的浏览器
JavaScript跟踪器在所有支持JSON API的浏览器上运行。这包括[IE8及更高版本](https://caniuse.com/?search=json)。单击此处以查看受支持的浏览器的完整列表。如果需要支持IE7和更早版本，则可以加载使window.JSON可用的polyfill，例如[JSON3](https://github.com/bestiejs/json3)。在加载Matomo JS跟踪器之前，需要先加载此polyfill。

### 已知的不兼容问题
原型js库将覆盖浏览器的JSON API，并导致问题，例如自定义变量导致错误（请参阅[＃16596](https://github.com/matomo-org/matomo/issues/16596)）。解决方法是删除原型JS库或手动覆盖JSON对象（请参见上文，例如使用JSON3）。

## JavaScript跟踪器功能
### 自定义页面名称
默认情况下，Matomo使用当前页面的URL作为Matomo报告中的页面标题。如果您的网址不简单，或者您想自定义Matomo跟踪页面的方式，则可以指定要在JavaScript代码中使用的页面标题。
一个常见的用例是将HTML页面的标题设置为文档标题：
```javascript
_paq.push(['setDocumentTitle', document.title]);
_paq.push(['trackPageView']);
```
如果您在同一个网站中*跟踪多个子域*，则可能希望页面标题以该子域为前缀，从而使您可以轻松查看每个子域的流量和数据。您可以简单地这样做：
```javascript
_paq.push(['setDocumentTitle', document.domain + "/" + document.title]);
_paq.push(['trackPageView']);
```
高级用户还可以例如使用PHP动态生成页面名称：
```php
_paq.push(['setDocumentTitle', "<?php echo $myPageTitle ?>"]);
_paq.push(['trackPageView']);
```
### 手动触发事件
默认情况下，当JavaScript跟踪代码在每个页面视图上加载并执行时，Matomo会跟踪页面视图。
但是，在现代Web应用程序上，用户交互不一定涉及加载新页面。例如，当用户单击JavaScript链接，单击选项卡（触发JS事件）或与用户界面元素进行交互时，您仍然可以跟踪与Matomo的交互。
要跟踪任何用户互动或与Matomo的点击，您可以手动调用JavaScript函数trackEvent（）。例如，如果要跟踪JavaScript菜单上的点击，则可以编写：
```html
<a href="#" onclick="_paq.push(['trackEvent', 'Menu', 'Freedom']);">Freedom page</a>
```
您可以在[用户指南](https://matomo.org/docs/event-tracking/#tracking-events)中了解有关跟踪事件的更多信息。
### 手动触发目标转化
默认情况下，Matomo中的目标定义为URL的“匹配”部分（以，包含或正则表达式匹配开头）。您还可以跟踪给定页面浏览量，下载量或外出点击量的目标。
在某些情况下，您可能希望在其他类型的操作上注册转换，例如：
- 用户提交表单时
- 当用户在页面上停留的时间超过给定时间时
- 用户在Flash应用程序中进行某些交互时
- 当用户提交购物车并完成付款时：您可以将Matomo跟踪代码提供给付款网站，然后该网站将转化记录在您的Matomo数据库中，并包含转化的自定义收入
要触发目标转化：
```javascript
// logs a conversion for goal 1
// //记录目标1的转化
_paq.push(['trackGoal', 1]);
```
您也可以使用自定义收入为该目标注册一次转化。例如，您可以动态生成对trackGoal()的调用以设置交易的收入:
```javascript
// logs a conversion for goal 1 with the custom revenue set
// 使用自定义收入集记录目标1的转化
_paq.push(['trackGoal', 1, <?php echo $cart->getCartValue(); ?>]);
```
在[“跟踪目标”](https://matomo.org/docs/tracking-goals-web-analytics/)文档中的Matomo中找到有关目标跟踪的更多信息。
### 准确衡量每页花费的时间
默认情况下，当用户在访问期间仅访问一个页面视图时，Matomo会假定访问者在网站上花费了0秒。这会带来一些后果：
- 当访问者仅查看一个页面视图时，“访问持续时间”将为0秒。
- 当访问者查看多个页面时，则访问的最后一个页面视图的“页面停留时间”为0秒。
可以配置Matomo，以便它可以准确地衡量访问所花费的时间。为了更好地衡量访问所花费的时间，请在JavaScript代码中添加以下内容：
```javascript
// accurately measure the time spent in the visit
// 准确衡量访问所花费的时间
_paq.push(['enableHeartBeatTimer']);
```
然后，只要用户正在积极查看页面（即当标签处于活动状态且处于焦点状态），Matomo就会发送请求，以计算访问的实际时间。在以下情况下执行心跳请求：
- 在当前标签页处于活动状态至少15秒钟（可进行配置，请参见下文）后，切换到另一个浏览器标签页。
- 导航到同一标签中的另一页。
- 关闭标签。
```
// Change how long a tab needs to be active to be counted as viewed in seconds
// 更改标签需要激活多长时间才能进行计数（以秒/秒为单位
// Requires a page to be actively viewed for 30 seconds for any heart beat request to be sent.
// 要求页面在30秒内处于活动状态，才能发送任何心跳请求
_paq.push(['enableHeartBeatTimer', 30]);
```
注意：在测试心跳计时器时，请记住确保浏览器选项卡具有焦点而不是焦点。开发人员工具或其他面板。
### 电子商务追踪
Matomo允许进行高级而强大的电子商务跟踪。请查阅[Ecommerce Analytics](https://matomo.org/docs/ecommerce-analytics/)文档，以获取有关电子商务报告以及如何设置电子商务跟踪的更多信息。
### 内部搜索跟踪
Matomo提供了高级[站点搜索](https://matomo.org/docs/site-search/)分析功能，可让您跟踪访问者如何使用内部网站搜索引擎。默认情况下，Matomo可以读取将包含搜索关键字的URL参数。但是，您也可以使用JavaScript函数`trackSiteSearch(...)`手动记录站点搜索关键字。
在您的网站的标准页面中，通常会通过`matomoTracker.trackPageView()`来调用以记录页面浏览量。在搜索结果页面上，您可以调用`piwikTracker.trackSiteSearch(keyword，category，searchCount)`函数来记录内部搜索请求。注意："关键字"参数是必需的，但category和searchCount是可选的。
```javascript
_paq.push(['trackSiteSearch',
    // Search keyword searched for
    // 搜索的搜索关键字
    "Banana",
    // Search category selected in your search engine. If you do not need this, set to false
    // 在搜索引擎中选择的搜索类别。如果不需要它，请设置为false
    "Organic Food",
    // Number of results on the Search results page. Zero indicates a 'No Result Search Keyword'. Set to false if you don't know
    // 搜索结果页面上的结果数。零表示“无结果搜索关键字”。如果您不知道，请设置为false
    0
]);

// We recommend not to call trackPageView() on the Site Search Result page
// 我们建议不要在“网站搜索结果”页面上调用trackPageView()
// _paq.push(['trackPageView']);
```
我们也强烈建议您设置searchCount参数，因为Matomo会具体报告“无结果关键字”。搜索过但没有返回任何结果的关键字。知道用户在搜索什么却在您的网站上找不到（还可以？）通常很有趣。在[用户文档](https://matomo.org/docs/site-search/)中了解有关网站搜索分析的更多信息。
### 自定义参数
自定义变量是一项强大的功能，可让您跟踪每次访问和/或每个页面视图的自定义值。请参阅[“跟踪自定义变量”](https://matomo.org/docs/custom-variables/)文档页面以获取常规信息。
您每次访问您的网站最多可以设置5个自定义变量（名称和值），和/或每个页面浏览量最多可以设置5个自定义变量。如果您为访问者设置了自定义变量，则当访客一小时或两天后返回时，将是一次新的访问，并且他/她的自定义变量将为空。
您可以将自定义变量设置为两个“作用域”。 “范围”是函数`setCustomVariable()`的第四个参数。
- `当scope = "visit"`时，自定义变量的名称和值将存储在访问数据库中。因此，您每次访问最多可以存储5个范围为"访问"的自定义变量。
- 当`scope = "page"`时，将为要跟踪的页面视图存储自定义变量的名称和值。因此，每个页面视图最多可以存储5个范围为"页面"的自定义变量。
"index"参数是自定义变量插槽索引，它是1到5之间的整数。（注意：如果您需要的不是默认的5个插槽，请阅读[此FAQ](https://matomo.org/faq/how-to/faq_17931/)）。
- 自定义变量统计信息在Matomo中的**访问者>自定义变量**下报告。范围"访问"和"页面"的两个自定义变量都在此报告中汇总。
### 自定义访问变量
```
setCustomVariable(index, name, value, scope = "visit")
```
此函数用于创建或更新自定义变量的名称和值。例如，假设您想在每次访问中存储用户的性别。您将使用name="gender"，value="male"或"female"存储自定义变量。
**重要说明**：给定的自定义变量名称必须始终存储在相同的"索引"中。例如，如果您选择将变量名 **name="Gender"** 存储在 **index= 1**中，并在**index=1**中记录另一个自定义变量，则"Gender"变量将被删除并替换为存储在索引1中的新自定义变量。
```
_paq.push(['setCustomVariable',
    // Index, the number from 1 to 5 where this custom 
    variable name is stored
    // 索引，此自定义变量名称存储在1到5之间的数字
    1,
    // Name, the name of the variable, for example: Gender, VisitorType
    //名称，变量的名称，例如：Gender，VisitorType
    "Gender",
    // Value, for example: "Male", "Female" or "new", "engaged", "customer"
    //值，例如："男"，"女"或"新"，"订婚"，"客户"
    "Male",
    // Scope of the custom variable, "visit" means the custom variable applies to the current visit
    // 自定义变量的范围，"visit"表示自定义变量适用于当前访问
    "visit"
]);

_paq.push(['trackPageView']);
```
您只需要设置一次范围为visit的变量，该值将记录在整个访问中。