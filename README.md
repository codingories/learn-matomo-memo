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
### 页面浏览量的自定义变量
```javascript
setCustomVariable(index, name, value, scope = "page")
```
除了跟踪"访问"的自定义变量外，有时分别跟踪每个页面视图的自定义变量也很有用。例如，对于"新闻"网站或博客，给定的文章可以分类为一个或几个类别。在这种情况下，您可以将一个或多个自定义变量的名称设置为name ="category"，如果将文章归类为"体育"和"欧洲类别"，则将一个值设置为 value = "Sports"，另一个将值设置为"value = Europe"。然后，自定义变量报告将报告您每个网站类别中的访问次数和页面浏览量。使用标准Matomo报告很难获得此信息，因为它们报告的"最佳页面URL"和"最佳页面标题"可能不包含"类别"信息。
```javascript
// Track 2 custom variables with the same name, but in different slots.
// 跟踪2个具有相同名称但位于不同插槽中的自定义变量。
// You will then access the statistics about your articles' categories in the 'Visitors &gt; custom variables' report
// 然后，您将在“访问者>自定义变量”报告中访问有关文章类别的统计信息
_paq.push(['setCustomVariable', 1, 'Category', 'Sports', 'page']);

// Track the same name but in a different Index
// 跟踪相同的名称但在不同的索引中
_paq.push(['setCustomVariable', 2, 'Category', 'Europe', 'page']);
// Here you could track other custom variables with scope "page" in Index 3, 4 or 5
// 在这里，您可以在索引3、4或5中跟踪范围为“ page”的其他自定义变量
// The order is important: first setCustomVariable is called and then trackPageView records the request
// 顺序很重要：首先调用setCustomVariable，然后trackPageView记录请求
_paq.push(['trackPageView']);
```
**重要提示**：可以在"索引"1​​中存储范围为"访问"的自定义变量，并在同一"索引" 1​​中存储范围为"页面"的不同自定义变量。因此，从技术上讲，您最多可以**跟踪10个自定义变量网站每个页面上**的变量名称和值（在实际页面视图中存储5个"页面"范围，在访问中存储5个"访问"范围）。
```
_paq.push(['setCustomVariable',
    // Index, the number from 1 to 5 where this custom variable name is stored for the current page view
    1,
    // 索引，从1到5的数字，此自定义变量名称存储在当前页面视图中
    // Name, the name of the variable, for example: Category, Sub-category, UserType
    // 名称，变量名称，例如：类别，子类别，用户类型
    "category",
    // Value, for example: "Sports", "News", "World", "Business", etc.
    // 值，例如："体育"，"新闻"，"世界"，"商业"等。
    "Sports",
    // Scope of the custom variable, "page" means the custom variable applies to the current page view
    // 自定义变量的范围，"页面"表示自定义变量适用于当前页面视图
    "page"
]);

_paq.push(['trackPageView']);
```
### 删除自定义变量
```javascript
deleteCustomVariable(index, scope)
```
如果创建了自定义变量，然后决定从访问或页面视图中删除此变量，则可以使用deleteCustomVariable。
要将更改保存在Matomo服务器中，必须在调用`trackPageView()`之前先调用该函数;
```javascript
_paq.push(['deleteCustomVariable', 1, "visit"]); 
// Delete the variable in index 1 stored for the current visit
// 删除索引1中为当前访问存储的变量
_paq.push(['trackPageView']);
```
### 检索自定义变量
```javascript
getCustomVariable(index, scope)
```
此函数可用于获取自定义变量的名称和值。默认情况下，它仅适用于在同一页面加载期间设置的自定义变量。
注意：可以配置Matomo，以便`getCustomVariable`也将返回范围为"visit"的自定义变量的名称和值，即使该变量是在同一访问的上一个网页浏览中设置的也是如此。要启用此行为，请在调用`trackPageView`之前调用JavaScript函数`storeCustomVariablesInCookie`。这将允许在第一方cookie中存储范围为"访问"的自定义变量。自定义变量Cookie在访问期间（最后一次操作后30分钟）有效。然后，您可以使用·getCustomVariable·检索自定义变量的名称和值。如果请求的索引中没有自定义变量，则它将返回false。
```javascript
_paq.push([function() {
    var customVariable = this.getCustomVariable( 1, "visit" );
    // Returns the custom variable: [ "gender", "male" ]
    // 返回自定义变量：["gender"，"male"]
    // do something with customVariable...
    // 用customVariable做某...
}]);
_paq.push(['trackPageView']);
```
### 自定义维度
[自定义维度](https://matomo.org/docs/custom-dimensions/)是一项强大的功能，可让您跟踪每次访问和/或每个操作（页面视图，链接，下载）的自定义值。 Matomo并未直接附带此功能，但可以通过Matomo Marketplace（CustomDimensions插件）作为插件安装。您必须先安装插件并配置至少一个维度，然后才能使用“自定义维度”，请参阅["自定义维度"](https://matomo.org/docs/custom-dimensions/)指南。您将为每个已配置的"自定义维度"获得一个数字ID，可用于为其设置值

### 跨跟踪请求跟踪自定义维度
要跟踪值，只需指定ID后跟一个值即可：
```javascript
_paq.push(['setCustomDimension', customDimensionId = 1, customDimensionValue = 'Member']);
```
请注意，一旦设置了"自定义维度"，该值将用于随后的所有跟踪请求，如果不希望这样做，可能会导致结果不准确。例如，如果您跟踪页面视图，则在同一页面加载中，针对以下每个事件，链接，下载等的"自定义维度"值也将被跟踪。调用此方法实际上不会触发跟踪请求，而是将这些值与以下跟踪请求一起发送。要在跟踪请求后删除自定义维度值，请调用`_paq.push（['deleteCustomDimension'，customDimensionId]`;
### 为初始页面视图设置自定义维度
要为初始页面视图设置自定义尺寸，请确保将方法调用setCustomDimension放置在trackPageView之前：
```javascript
_paq.push(['setCustomDimension', customDimensionId = 1, customDimensionValue = 'Member']);
_paq.push(['trackPageView']);
// _paq.push(['enableLinkTracking']);
// rest of tracking code
// 其余跟踪代码
```
#### 仅跟踪一项特定操作的自定义维度
只能为一个特定操作设置自定义维度。如果要跟踪页面视图，可以将一个或多个特定的“自定义维度”值与此跟踪请求一起发送，如下所示：
`_paq.push(['trackPageView', pageTitle, {dimension1: 'DimensionValue'}]);`
要定义维度值，请通过一个对象，该对象将一个或多个属性定义为最后一个参数（确保指定方法中定义的所有参数，我们不会自动假定最后一个参数为customData，而是方法定义的所有参数都需要传递给每种方法）。维度的属性名称以维度开头，后跟自定义维度ID，例如Dimension1。相同的行为适用于其他几种方法：
```javascript
_paq.push(['trackEvent', category, action, name, value, {dimension1: 'DimensionValue'}]);
_paq.push(['trackSiteSearch', keyword, category, resultsCount, {dimension1: 'DimensionValue'}]);
_paq.push(['trackLink', url, linkType, {dimension1: 'DimensionValue'}]);
_paq.push(['trackGoal', idGoal, customRevenue, {dimension1: 'DimensionValue'}]);
```
优点是，设置的维度值将仅用于此特定操作，并且您不必在跟踪请求后删除该值。您可以这样设置多个维度值：
`_paq.push(['trackPageView', pageTitle, {dimension1: 'DimensionValue', dimension4: 'Test', dimension7: 'Value'}]);`
### 检索自定义维度值
```javascript
getCustomDimension(customDimensionId)
```
此函数可用于获取自定义维度的值。仅当在同一页面加载期间设置了"自定义维度"时，此功能才有效。
### 用户身份
[用户ID](https://matomo.org/docs/user-id/)是Matomo中的一项功能，可让您将从多个设备和多个浏览器收集的给定用户的数据连接在一起。实施用户ID有两个步骤:
- 您必须分配一个代表每个登录用户的唯一且持久的非空字符串。通常，此ID是您的身份验证系统提供的电子邮件地址或用户名。
- 您必须为每个综合浏览量设置用户ID，否则将在不设置用户ID的情况下跟踪综合浏览量。
- 然后，您必须在调用任何track *函数（trackPageview，trackEvent，trackGoal，trackSiteSearch等）之前，通过setUserId方法调用将此用户ID字符串传递给Matomo。
```javascript
_paq.push(['setUserId', 'USER_ID_HERE']);
_paq.push(['trackPageView']);
```
注意：USER_ID_HERE必须是一个唯一且持久的非空字符串，表示跨设备的用户。
### 用户登录后，设置用户ID
让我们举个例子。假设您的网站使用PHP脚本通过登录表单对用户进行身份验证。 Matomo JavaScript代码段如下所示：
```php
var _paq = window._paq = window._paq || [];
<?php
// If user is logged-in then call 'setUserId'
// 如果用户已登录，则调用"setUserId"
// 用户成功通过您的应用程序身份验证后，服务器必须设置$ userId变量。
if (isset($userId)) {
     echo sprintf("_paq.push(['setUserId', '%s']);", $userId);
}
?>

_paq.push(['trackPageView']);
_paq.push(['enableLinkTracking']);
```
### 用户注销后，重置用户标识
当用户注销并且用户ID不再可用时，建议通过在trackPageView之前调用resetUserId方法来通知Matomo。
如果您要在用户注销时创建新的访问，则还可以通过以下方法，通过调用resetUserId和appendToTrackingUrl（两次）来强制Matomo创建新的访问：
```javascript
// User has just logged out, we reset the User ID
// 用户刚刚注销，我们重置用户ID
_paq.push(['resetUserId']);

// we also force a new visit to be created for the pageviews after logout
// 我们也会在登出后强制为浏览量创建新的访问
_paq.push(['appendToTrackingUrl', 'new_visit=1']); 
_paq.push(['trackPageView']);
// we finally make sure to not again create a new visit afterwards (important for Single Page Applications)
// 我们最后确保不要再创建新的访问（对于单页应用程序很重要）
_paq.push(['appendToTrackingUrl', '']); 
```
### 内容追踪
有几种方法可以手动，半自动和自动地跟踪内容印象和交互。请注意，即使默认配置为GET，也将使用批量跟踪来跟踪内容展示，批量跟踪将始终发送POST请求。有关更多详细信息，请参阅内容跟踪的[深入指南](https://developer.matomo.org/guides/content-tracking)。

### 跟踪页面中的所有内容印象
您可以使用`trackAllContentImpressions()`方法来扫描整个DOM中的内容块。对于每个内容块，我们将立即跟踪内容印象。如果只想跟踪可见内容的印象，请查看`trackVisibleContentImpressions()`。
```javascript
_paq.push(['trackPageView']);
_paq.push(['trackAllContentImpressions']);
```
如果您多次调用此方法，则不会两次发送相同内容块的印象，除非同时调用trackPageView（）。这对于单页应用程序很有用。

### 仅跟踪页面内可见的内容印象。
启用仅通过`trackVisibleContentImpressions（checkOnScroll，timeIntervalInMs）`跟踪可见内容的印象。 “可见”是指内容块已位于视口中并且未被隐藏（opacity, visibility, display, ...）。
  - （可选）您可以通过传递checkOnScroll = false告诉我们在每次滚动后不要重新扫描DOM。否则，我们将检查之前隐藏的内容块是否在滚动之后同时可见，如果是，则跟踪印象。
    - 限制：如果将内容块放置在可滚动元素内（溢出：滚动），则我们目前无法检测到该元素何时可见。
  - （可选）您可以告诉我们通过传递timeIntervalInMs = 500每隔X毫秒重新扫描整个DOM，以获得新的内容展示。默认情况下，我们将每750毫秒重新扫描一次DOM。要禁用它，请传递timeIntervalInMs = 0。
    - 重新扫描整个DOM并检测内容块的可见状态可能需要一段时间，具体取决于浏览器，硬件和内容量。万一每秒的帧数下降，您可能需要增加间隔或完全禁用间隔。如果禁用它，您仍然可以随时通过再次调用此方法或trackContentImpressionsWithinNode（）手动重新扫描DOM。
首次调用此方法后，不能同时更改checkOnScroll和timeIntervalInMs。
```javascript
_paq.push(['trackPageView']);
_paq.push(['trackVisibleContentImpressions', true, 750]);
```
### 仅跟踪页面一部分的内容展示
如果在我们跟踪初始印象后要向DOM中添加元素，请使用trackContentImpressionsWithinNode（domNode，contentTarget）方法。调用此方法将确保跟踪此节点中包含的所有内容块的印象。
例子:
```javascript
var div = $('<div>...<div data-track-content>...</div>...<div data-track-content>...</div></div>');
$('#id').append(div);

_paq.push(['trackContentImpressionsWithinNode', div[0]]);
```
在此示例中，我们将检测到两个新的内容印象。如果您仅启用了跟踪可见的内容块，我们将考虑到这一点。
### 半自动跟踪互动
通常，一旦访问者单击内容块，就会自动跟踪与内容块的交互。有时，例如，如果您想基于表单提交或双击来触发交互，则可能需要手动触发交互。为此，调用方法trackContentInteractionNode（domNode，contentInteraction）。
例:
```javascript
formElement.addEventListener('submit', function () {
    _paq.push(['trackContentInteractionNode', this, 'submittedForm']);
});
```
  - 传递的`domNode`可以是内容块内的任何节点，也可以是内容块元素本身。万一找不到内容块，将不会追踪任何内容。
  - （可选）您可以设置内容交互的名称，例如单击或提交。如果未提供任何值，则将使用值Unknown。
  - 您应该通过设置CSS类piwikContentIgnoreInteraction或属性data-content-ignoreinteraction来禁用对该内容块的自动交互跟踪。否则，一旦访问者单击，就可能在其顶部跟踪交互。
当您手动触发交互时，我们将这种跟踪称为半自动跟踪，但是会自动检测到内容名称，片段和目标。自动检测内容名称和内容可确保我们可以将互动与以前跟踪的印象进行映射。

### 手动跟踪内容展示和互动
您仅应一起使用方法trackContentImpression（contentName，contentPiece，contentTarget）和trackContentInteraction（contentInteraction，contentName，contentPiece，contentTarget）。不建议您在自动跟踪印象之后使用trackContentInteraction（），因为只有在您设置与跟踪相关印象相同的内容名称和内容时，我们才能将交互映射到印象。
例:
```javascript
_paq.push(['trackContentImpression', 'Content Name', 'Content Piece', 'https://www.example.com']);

div.addEventListener('click', function () {
    _paq.push(['trackContentInteraction', 'tabActivated', 'Content Name', 'Content Piece', 'https://www.example.com']);
});
```
请注意，对这些方法的每次调用都会向您的Matomo跟踪器实例发送一个请求。重复执行多次可能会导致性能问题。
## 检测域名和/或子域名
无论您要跟踪一个域，一个子域，还是同时跟踪两者，等等，都可能需要配置Matomo JavaScript跟踪代码。可能需要配置两件事：1）如何创建和共享跟踪cookie，以及2）应该将哪些点击作为"外部链接"进行跟踪。

### 跟踪一个域名
这是标准用例。 Matomo可以在一个Matomo网站中跟踪一个没有子域的域名的访问。
```javascript
// Default Tracking code
_paq.push(['setSiteId', 1]);
_paq.push(['setTrackerUrl', u+'matomo.php']);
_paq.push(['trackPageView']);
```
如果您要跟踪一个特定的子域，则此默认跟踪代码也可以使用。
### 在同一网站中跟踪一个域及其子域
要记录主域名及其子域中的所有用户，我们告诉Matomo在所有子域中共享cookie。在example.com/*和所有子域的Matomo跟踪代码中调用setCookieDomain（）。
```javascript
_paq.push(['setSiteId', 1]);
_paq.push(['setTrackerUrl', u+'matomo.php']);

// Share the tracking cookie across example.com, www.example.com, subdomain.example.com, ...
// 在example.com，www.example.com，subdomain.example.com，...之间共享跟踪Cookie
_paq.push(['setCookieDomain', '*.example.com']);

// Tell Matomo the website domain so that clicks on these domains are not tracked as 'Outlinks'
//告诉Matomo网站域，这样就不会将这些域的点击跟踪为"链接"
_paq.push(['setDomains', '*.example.com']);

_paq.push(['trackPageView']);
```
### 在同一个网站中跨多个域名跟踪访客
为了准确地将跨不同域名的访问者跟踪到一个Matomo网站中的单次访问中，我们需要设置所谓的跨域链接。 Matomo中的跨域跟踪可确保当访问者访问多个网站和域名时，访问者数据将存储在同一访问中，并且访问者ID可在各个域名之间重复使用。例如，当电子商务在线商店位于www.awesome-shop.com上，而电子商务购物车技术位于另一个域（如secure.cart.com）上时，则需要跨域的典型用例
跨域链接结合使用了两种跟踪器方法setDomains和enableCrossDomainLinking。在我们的指南中了解如何设置跨域链接：[如何跨多个域名准确衡量同一个访问者（跨域链接）？](https://matomo.org/faq/how-to/faq_23654/)

### 在单独的网站中跟踪域的子目录
在各自独立的Matomo网站中跟踪域的子目录时，建议自定义跟踪代码，以确保最佳的数据准确性和性能。
例如，如果您的网站提供"用户个人资料"功能，则您可能希望在Matomo的单独网站中跟踪每个用户个人资料页面。在主域主页中，您将使用默认跟踪代码：
```javascript
// idSite = X for the Homepage
// idSite = X作为主页
// In Administration > Websites for idSite=X, the URL is set to `example.com/`
//在管理> idSite = X的网站中，URL设置为`example.com/
_paq.push(['setSiteId', X]);
_paq.push(['setTrackerUrl', u+'matomo.php']);
_paq.push(['trackPageView']);
```
在`example.com/user/MyUsername`页面（以及所有其他用户配置文件）中，您将构造对自定义`setSiteId`，`setCookiePath`和`setDomains`的调用：
```javascript
// The idSite Y will be different from other user pages
// idSite Y将与其他用户页面不同
// In Administration > Websites for idSite=Y, the URL is set to `example.com/user/MyUsername`
// 在管理> idSite = Y的网站中，URL设置为`example.com / user / MyUsername`。
_paq.push(['setSiteId', Y]);

// Create the tracking cookie specifically in `example.com/user/MyUsername`
//在`example.com / user / MyUsername`中专门创建跟踪cookie
_paq.push(['setCookiePath', '/user/MyUsername']);

// Tell Matomo the website domain so that clicks on other pages (eg. /user/AnotherUsername) will be tracked as 'Outlinks'
//告诉Matomo网站域名，以便将其他页面（例如/ user / AnotherUsername）上的点击记录为"外部链接"
_paq.push(['setDomains', 'example.com/user/MyUsername']);

_paq.push(['setTrackerUrl', u+'matomo.php']);
_paq.push(['trackPageView']);
```
在单独的网站中跟踪许多子目录时，函数`setCookiePath`阻止cookie的数量快速增加，并阻止浏览器删除某些cookie。这样可以确保最佳的数据准确性并提高用户的性能（每次请求发送的Cookie减少）。
functionsetDomains确保将离开您网站（子目录example.com/user/MyUsername）的用户的点击正确跟踪为"外接"。
### 在单独的网站中跟踪一组页面
（自Matomo 2.16.1开始可用）
在极少数情况下，您可能希望跟踪特定网站中与通配符匹配的所有页面，并将其他页面（不匹配通配符）上的点击跟踪为"外部链接"。
在/index_fr.htm或/index_en.htm页面中，输入：
```javascript
// clicks on links not starting with example.com/index will be tracked as 'Outlinks'
//点击不是以example.com/index开头的链接的情况将被跟踪为“外部链接”
_paq.push(['setDomains', 'example.com/index*']);
// when using a wildcard *, we do not need to configure cookies with `setCookieDomain`
//使用通配符*时，我们不需要使用`setCookieDomain`配置cookie
// or `setCookiePath` as cookies are correctly created in the main domain by default
//或`setCookiePath`，因为默认情况下会在主域中正确创建cookie
_paq.push(['setTrackerUrl', u+'matomo.php']);
_paq.push(['trackPageView']);
```
笔记:
  - 仅在字符串末尾指定通配符*时才受支持。
  - 由于通配符可以匹配多个路径，因此将省略对setCookieDomain或setCookiePath的调用，以确保为与通配符匹配的所有页面正确共享跟踪cookie。
有关在Matomo中跟踪网站和子域的更多信息，请参阅FAQ：[如何配置Matomo来监视多个网站，域和子域](https://matomo.org/faq/new-to-piwik/#faq_104)
## 下载和出站跟踪
### 启用下载和出站跟踪
默认的Matomo JavaScript跟踪器代码会自动启用下载和出站跟踪，方法是调用enableLinkTracking函数来完成：
```javascript
// Enable Download & Outlink tracking
// 启用下载和链接跟踪
_paq.push(['enableLinkTracking']);
```
建议在第一次调用trackPageView或trackEvent之后添加此行。

### 自动跟踪外链
默认情况下，除当前域外，所有指向其他域的链接都启用了点击跟踪，每次点击都将被计为出站链接。如果您使用多个域和子域，则可能会在页面>出站报告中看到对子域的点击。
### 跟踪出站并忽略别名域
如果只希望对外部网站的点击显示在“出站报告”中，则可以使用setDomains（）函数指定别名域或子域的列表。支持通配符域（* .example.org），可让您轻松忽略对所有子域的点击。
```javascript
// Don't track Outlinks on all clicks pointing to *.hostname1.com or *.hostname2.com
// 不要在指向* .hostname1.com / product1 / *或* .hostname2.com / product1 / *的所有点击上跟踪出站链接
// Note: the currently tracked website is added to this array automatically
// 注意：当前跟踪的网站会自动添加到此数组中
_paq(['setDomains', ["*.hostname1.com", "hostname2.com"]]);
_paq.push(['trackPageView']);
```
从Matomo 2.15.1开始，您还可以将路径附加到域，并且Matomo将正确地检测到相同域但链接与出站路径不同的链接。
```javascript
// Don't track Outlinks on all clicks pointing to *.hostname1.com/product1/* or *.hostname2.com/product1/*
// 不要在指向* .hostname1.com / product1 / *或* .hostname2.com / product1 / *的所有点击上跟踪出站链接
// Track all clicks not pointing to *.hostname1.com/product1/* or *.hostname2.com/product1/* as outlink.
// 跟踪所有未指向* .hostname1.com / product1 / *或* .hostname2.com / product1 / *作为链接的点击。
_paq(['setDomains', ["*.hostname1.com/product1", "hostname2.com/product1"]]);
```
了解有关此用例的更多信息在单独的网站中跟[踪域的子目录](https://developer.matomo.org/guides/tracking-javascript-guide#tracking-subdirectories-of-a-domain-in-separate-websites)。

### 通过CSS或JavaScript将点击作为链接进行跟踪
如果要强制Matomo将链接视为出站链接（链接到当前域或别名域之一），则可以将'​​piwik_link'css类添加到链接中：
```javascript
<a href='https://mysite.com/partner/' class='piwik_link'>Link I want to track as an outlink</a>
```
注意：您可以自定义和重命名用于强制将点击记录为链接的CSS类：
```javascript
// now all clicks on links with the css class "external" will be counted as outlinks
// 现在，所有对css类为“外部”的链接的点击都将计为出站链接
// you can also pass an array of strings
// 您还可以传递字符串数组
_paq.push(['setLinkClasses', "external"]);

_paq.push(['trackPageView']);
```
或者，您可以使用JavaScript手动触发对链接的单击（对于页面浏览或文件下载，其作用相同）。在此示例中，单击电子邮件地址时将触发自定义出站链接：
```javascript
<a href="mailto:namexyz@mydomain.co.uk" target="_blank" onClick="_paq.push(['trackLink', 'https://mydomain.co.uk/mailto/Agent namexyz', 'link']);">namexyz@mydomain.co.uk </a>
```
### 跟踪文件下载
默认情况下，任何以这些扩展名之一结尾的文件在Matomo界面中都将被视为"下载"：
```javascript
7z|aac|arc|arj|apk|asf|asx|avi|bin|bz|bz2|csv|deb|dmg|doc|
exe|flv|gif|gz|gzip|hqx|jar|jpg|jpeg|js|mp2|mp3|mp4|mpg|
mpeg|mov|movie|msi|msp|odb|odf|odg|odp|ods|odt|ogg|ogv|
pdf|phps|png|ppt|qt|qtm|ra|ram|rar|rpm|sea|sit|tar|
tbz|tbz2|tgz|torrent|txt|wav|wma|wmv|wpd||xls|xml|z|zip
```
### 自定义跟踪下载的文件类型
要替换文件下载时要跟踪的扩展名列表，可以使用`setDownloadExtensions（string）`：
```javascript
// we now only track clicks on images
// 现在，我们仅跟踪图片的点击次数
_paq.push(['setDownloadExtensions', "jpg|png|gif"]);

_paq.push(['trackPageView']);
```
如果要跟踪新文件类型，可以使用addDownloadExtensions（filetype）将其添加到列表中：
```javascript
// clicks on URLs finishing by mp5 or mp6 will be counted as downloads
//点击以mp5或mp6结尾的网址将被视为下载
_paq.push(['addDownloadExtensions', "mp5|mp6"]);
_paq.push(['trackPageView']);
```
如果要忽略特殊的文件类型，可以使用`removeDownloadExtensions（filetype）`将其从列表中删除：
```javascript
// clicks on URLs ending in png or mp4 will not be counted as downloads
_paq.push(['removeDownloadExtensions', "png|mp4"]);
_paq.push(['trackPageView']);
```
### 将点击记录为下载
如果要强制Matomo将链接视为下载，可以将matomo_download或piwik_download css类添加到链接：
```javascript
<a href='last.php' class='matomo_download'>Link I want to track as a download</a>
```
注意：您可以自定义并重命名用于强制将点击记录为下载的CSS类：
```javascript
// now all clicks on links with the css class "download" will be counted as downloads
// 现在，所有具有css类“下载”的链接的点击都将计为下载
// you can also pass an array of strings
// 您还可以传递字符串数组
_paq.push(['setDownloadClasses', "download"]);

_paq.push(['trackPageView']);
```
另外，您可以使用JavaScript手动触发对下载的点击。在此示例中，单击链接会触发自定义下载：
```javascript
<a href="https://secure.example.com/this-is-a-file-url" target="_blank" onClick="_paq.push(['trackLink', 'https://mydomain.co.uk/mailto/Agent namexyz', 'download']);">Download</a>
```
### 更改暂停计时器
当用户单击下载文件或单击出站链接时，Matomo会记录该文件。为此，在用户重定向到所请求的文件或链接之前，它会增加一小段延迟。默认值为500ms，但是您可以将其设置为较短的时间长度。但是，应注意的是，这样做会导致以下风险：该时间段不足以将数据记录在Matomo中。
```javascript
_paq.push(['setLinkTrackingTimer', 250]); 
// 250 milliseconds
// 毫秒
_paq.push(['trackPageView']);
```
### 禁用下载和链接跟踪
默认情况下，Matomo跟踪代码启用点击和下载跟踪。要禁用所有自动下载和出站跟踪，必须删除对enableLinkTracking（）函数的调用：
```javascript
_paq.push(['trackPageView']);

// we comment out the function that enables link tracking
// 我们注释掉了启用链接跟踪的功能
// _paq.push(['enableLinkTracking']);
```
### 禁用特定的CSS类
您可以对具有特定CSS类的链接禁用自动下载和出站跟踪：
```javascript
// you can also pass an array of strings
// 您还可以传递字符串数组
_paq.push(['setIgnoreClasses', "no-tracking"]);
_paq.push(['trackPageView']);
```
这将导致不计算链接`<a href='https://example.com' class='no-tracking'>测试</a>`上的点击。
### 禁用特定链接
如果要忽略特定链接上的下载或外链跟踪，可以向其添加`matomo_ignore`或`piwik_ignore`css类：
```html
<a href='https://builds.matomo.org/latest.zip' class='matomo_ignore'>File I don't want to track as a download</a>
```
## 征求同意
[查看我们的集成指南以实施跟踪或Cookie同意。](https://developer.matomo.org/guides/tracking-consent)
### 可选：创建自定义退出表单
如果您想为您的用户提供完全退出跟踪的功能，则可以使用退出表单。 Matomo随附了使用第三方Cookie的退出表单实现（您可以在Matomo中的“ Matomo”>“管理”>“隐私”页面上对其进行配置）。
嵌入此表单很简单，因为它只需要您向网站添加iframe，但并不总是理想的。一些用户阻止了第三方Cookie，因此退出表单对他们不起作用。您可能还希望以“退出”表单显示自定义文本或图形，或者可能希望允许用户单独而不是完全退出站点。
在这种情况下，您可能需要考虑创建自定义退出表单。创建HTML / JS表单的细节超出了本文档的范围，但是每个自定义退出表单都需要做一些事情：检查用户当前是否退出，用户并退出用户。这是如何做这些事情：
检查用户当前是否选择退出
使用isUserOptedOut（）方法，如下所示：
_paq.push([function () {
  if (this.isUserOptedOut()) {
    // ... change form to say user is currently opted out ...
    // ...更改表格以说用户当前已退出...
  } else {
    // ... change form to say user is currently opted in ...
    // ...更改表格以说用户当前选择了...
  }
}])
选择退出用户
使用optUserOut（）方法：
_paq.push(['optUserOut']);
选择一个用户
使用forgetUserOptOut（）方法：
```
_paq.push(['forgetUserOptOut']);
```
下面是一个示例退出表单，该表单复制了内置的Matomo退出表单：

```
<div id="optout-form">
  <p>You may choose not to have a unique web analytics cookie identification number assigned to your computer to avoid the aggregation and analysis of data collected on this website.</p>
  <p>您可以选择不为计算机分配唯一的网络分析cookie标识号，以避免对本网站收集的数据进行汇总和分析。
  <p>To make that choice, please click below to receive an opt-out cookie.</p>
  <p>要做出选择，请单击下面的按钮，以接收退出cookie。</ p>
  <p>
    <input type="checkbox" id="optout" />
    <label for="optout"><strong></strong></label>
  </p>
</div>
<script>
document.addEventListener("DOMContentLoaded", function(event) {
  function setOptOutText(element) {
    _paq.push([function() {
      element.checked = !this.isUserOptedOut();
      document.querySelector('label[for=optout] strong').innerText = this.isUserOptedOut()
        ? 'You are currently opted out. Click here to opt in.'
        : 'You are currently opted in. Click here to opt out.';
    }]);
  }

  var optOut = document.getElementById("optout");
  optOut.addEventListener("click", function() {
    if (this.checked) {
      _paq.push(['forgetUserOptOut']);
    } else {
      _paq.push(['optUserOut']);
    }
    setOptOutText(optOut);
  });
  setOptOutText(optOut);
});
</script>

```
## 多个Matomo追踪器
默认情况下，Matomo JavaScript跟踪代码将您的分析数据收集到一台Matomo服务器中。 Matomo服务网址是在您的JavaScript跟踪代码中指定的（例如：var u =“ // matomo.example.org”;）。在某些情况下，您可能希望将分析数据跟踪到多个Matomo服务器或同一Matomo服务器上的多个网站中。
如果尚未升级到Matomo 2.16.2或更高版本，请立即升级！ （以下是2.16.1或更早版本的说明。）
### 将您的数据复制到一台Matomo服务器中的不同网站中
您可能需要将Web分析数据的副本收集到相同的Matomo服务器中，但是在另一个网站中。
### 推荐的解决方案：使用汇总报告插件
当您需要将数据复制到另一个网站，或将多个网站合并为一个或多个组（称为汇总）时，建议的解决方案是使用[汇总报告高级插件](https://plugins.matomo.org/RollUpReporting)。与其他解决方案相比，使用此插件具有多个优点，因为您可以轻松地将一个或多个网站组合在一起，并且汇总不会导致重复跟踪数据，从而提高了整体性能。
### 替代解决方案：复制跟踪数据
除了使用汇总报告插件外，您还可以复制跟踪数据。要复制数据，您可以调用带有Matomo URL的addTracker以及您在其中复制数据的网站ID：
```javascript
var u="//matomo.example.org/";
  _paq.push(['setTrackerUrl', u+'matomo.php']);
  _paq.push(['setSiteId', '1']);

  // We will also collect the website data into Website ID = 7
  //我们还将网站数据收集到网站ID = 7中
  var websiteIdDuplicate = 7;
  // The data will be duplicated into `piwik.example.org/matomo.php`
  //数据将被复制到`piwik.example.org / matomo.php`中
  _paq.push(['addTracker', u+'matomo.php', websiteIdDuplicate]);
  // Your data is now tracked in both website ID 1 and website 7 into your piwik.example.org server!
  //现在，您在网站ID 1和网站7中都可以跟踪您的数据到piwik.example.org服务器中！
```
由于此解决方案会导致在Matomo服务器中对每个访问者的事件，综合浏览量等进行两次跟踪，因此通常不建议这样做。
### 将您的分析数据收集到两个或更多Matomo服务器中
下面的示例显示了如何使用addTracker方法将相同的分析数据跟踪到第二个Matomo服务器中。 Matomo主服务器是piwik.example.org/matomo.php，其中数据存储在网站ID 1中。第二个Matomo服务器是analytics.example.com/matomo.php，其中数据存储在网站ID 77中。在您的网站中实施此操作，请用您自己的Matomo URL和网站ID替换这两个Matomo URL和Matomo网站ID。
```javascript
<script type="text/javascript">
  var _paq = window._paq = window._paq || [];
  _paq.push(['trackPageView']);
  _paq.push(['enableLinkTracking']);

  (function() {
    var u="//matomo.example.org/";
    _paq.push(['setTrackerUrl', u+'matomo.php']);
    _paq.push(['setSiteId', '1']);

    // Add this code below within the Matomo JavaScript tracker code
    //将此代码添加到Matomo JavaScript跟踪器代码的下面
    // Important: the tracker url includes the /matomo.php
    // 重要提示：跟踪器网址包含/matomo.php
    var secondaryTrackerUrl = 'https://analytics.example.com/matomo.php';
    var secondaryWebsiteId = 77;
    // Also send all of the tracking data to this other Matomo server, in website ID 77
    // 还将所有跟踪数据发送到网站ID为77的另一个Matomo服务器
    _paq.push(['addTracker', secondaryTrackerUrl, secondaryWebsiteId]);
    // That's it!

    var d=document, g=d.createElement('script'), s=d.getElementsByTagName('script')[0];
    g.type='text/javascript'; g.async=true; g.src=u+'matomo.js'; s.parentNode.insertBefore(g,s);
  })();
</script>

```
### 自定义其中一个跟踪器对象实例
注意：默认情况下，通过addTracker添加的任何跟踪器的配置都与主要的默认跟踪器对象相同（关于cookie，自定义维度，用户ID，下载和链接跟踪，域和子域等）。如果要配置通过addTracker添加的Matomo跟踪器对象实例之一，则可以调用Matomo.getAsyncTracker（optionalMatomoUrl，optionalPiwikSiteId）方法。此方法返回的跟踪器实例对象可以与主要的JavaScript跟踪器对象实例进行不同的配置。
### 直接调用JavaScript API（而不是通过_paq.push）时复制跟踪数据
可以将分析数据跟踪到同一服务器上的其他网站ID中，也可以将数据的副本记录到另一台Matomo服务器中。每次调用Piwik.getTracker（）都会返回一个可以配置的唯一Matomo Tracker对象（实例）。
```javascript
<script type="text/javascript">
    window.matomoAsyncInit = function () {
        try {
            var matomoTracker = Matomo.getTracker("https://URL_1/matomo.php", 1);
            matomoTracker.trackPageView();
            var piwik2 = Matomo.getTracker("https://URL_2/matomo.php", 4);
            piwik2.trackPageView();
        } catch( err ) {}
    };
</script>
```
一旦加载并初始化Matomo跟踪器，就会执行matomoAsyncInit（）方法。在早期版本中，您必须加载Matomo sync。
### JavaScript Tracker参考
在[JavaScript Tracker](https://developer.matomo.org/guides/tracking-javascript)参考中查看Tracking客户端的所有功能。
### 常问问题
如果您对Matomo中的JavaScript跟踪有任何疑问，请[搜索网站](https://matomo.org/)或在[论坛](https://forum.matomo.org/)中提问。
- [如何为没有JavaScript的用户启用跟踪？](https://matomo.org/faq/how-to/faq_176/)
- [Matomo如何跟踪下载？](https://matomo.org/faq/new-to-piwik/faq_47/)
- [如何跟踪单页网站和Web应用程序](https://developer.matomo.org/guides/spa-tracking)
- [如何跟踪错误页面并获取404和引荐来源网址列表。](https://matomo.org/faq/how-to/faq_60/)
- [如何设置自定义页面组（结构），以便按类别汇总页面视图？](https://matomo.org/faq/how-to/faq_62/)
- [如何设置Matomo以跟踪多个网站，而又不透露JS中的Matomo服务器URL足迹？](https://matomo.org/faq/how-to/faq_132/)
- [如何自定义在所有网站上加载的matomo.js？](https://matomo.org/faq/how-to/faq_19087/)
- [如何在JavaScript代码中禁用Matomo使用的所有跟踪Cookie？](https://matomo.org/faq/how-to/faq_19087/)





































