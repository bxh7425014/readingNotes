[焦英俊]-阿里巴巴店铺的十年技术演变之路

[「原文链接」](http://mp.weixin.qq.com/s?__biz=MzA5Nzc4OTA1Mw==&mid=2659598195&idx=1&sn=a69628c628c474d1b1368d139d64714f&scene=1&srcid=0920jq5fek5TugbDJN78ki88#rd)

阿里巴巴所提倡的 “大中台，小前台”战略是非常先进的业务打法，这个策略对技术的挑战其实更大！

文章作者是全程网络CTO焦英俊，前阿里巴巴资深技术专家，1688总架构&垂直开放平台负责人&云电商平台创始人。

阿里店铺到10年经历了4个大的主版本。

![阿里店铺到10年经历了4个大的主版本](http://upload-images.jianshu.io/upload_images/1124873-a0b30e9a9715a386.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 店铺系统的挑战
挑战划分为三大维度的问题：全平台、垂直化、个性化。

- 全平台：PC端、App端、Wap端，三端统一。我来举个实际例子，某一天业务人员找到技术说下个月要搞促销，我们希望在指定一批商家的店铺上投放特定优惠券，希望每个端都能同步投放，怎么破？

- 垂直化：电商领域其实非常广泛，你不可能要求做原材料的行业和做快消品的行业是一样的体验。那会出问题，举个例子，原材料就是需要大量的规格展示，越专业越好；快消品主打营销，希望更多的营销主题；甚至某些企业还想提供独立的官网来提供企业的知名度。**面对不同行业，不同诉求，这势必要求系统具备垂直化的能力来应对。**

- 个性化：这个更容易理解了，每个商家都希望自己的店铺和其他人是不一样的，都有自己的特色！千人千面。

# 店铺的中台架构策略

**模块化策略**
首先，我们一起看下这张图片，大家应该都非常熟悉。商品陈列信息，阿里内部称为offer橱窗。

![offer橱窗](http://upload-images.jianshu.io/upload_images/1124873-71dadb366304c7d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

逛过电商网站的同学应该知道，这么个看似简单的内容，会有非常多的呈现方式，常规的都能数上很多，更不用说个性化的方式。

粗略的分析一下：
- 页面元素：图片、价格、订购

- 数据元素：大图（需要筛选出来合适的尺寸，阿里为了图片的美观做了多次的优化，目前可以做到按需加载不同尺寸的图片）、价格（是否有活动、折扣、会员价）、订购按钮（有没有权限，有没有库存）

- 显示要求：平铺

从这三点要求上来看，可以看出，这个小板块背后的逻辑还是比较复杂的。以前我们怎么做？

第一个版本：硬编码，开发写了一个web control里面调用各种service，模板也单独写了一个（那个恐怖的时代，我印象中，页面超级复杂，大量的页面、JavaScript逻辑是copy出来的，轻易不敢维护），这个版本应该持续了有5、6年，相当痛苦。

第二个版本：control层widget，当时的架构师花了大量的时间构建了基于widget模式的系统。后端的逻辑看起来优雅了下一些，但是前端依然是全部写在一个页面当中的。当时web的同学最大的担忧就是发布，一个小小的VM都能引发一个p1故障。这个状态大概持续了3年左右。

第三个版本：组件化，页面层的种种问题积累了太多的血淋淋的教训！当时团队实在忍受不了了，决定全面实施组件化，**加强了前端层面分组件开发的模式。**这个是个巨大的进步，但是没有持续到半年就发现新问题了！整个系统设计的复杂度非常庞大（学院模式的过度设计），加上阿里惯性的拥抱变化，最后无人可维护。同时没有办法支持多个站点的需求，只能解决单一店铺。这个版本最后持续了一年多的时间。

第四个版本：模块化！简单的来描述就是**前端组件化＋后端组件化，当然这里需要遵循同一个组件协议！**开发人员也无需关心是在前端渲染还是后端渲染，这里最重要的点是让前后一致了，下面我们再具体介绍一下。

# “前后端一致”的组件化

![App全新概念](http://upload-images.jianshu.io/upload_images/1124873-b89e387d6b62c73d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![页面结构](http://upload-images.jianshu.io/upload_images/1124873-c3a47ad117307133.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除了布局，当然还有一些其他技术的细节，比如校验、session管理、异步通信等，这里不再描述。最后看一个页面的案例。

![页面案例](http://upload-images.jianshu.io/upload_images/1124873-d8ecad8a3dc577b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 统一数据中心

![统一数据中心](http://upload-images.jianshu.io/upload_images/1124873-0fd2267ef4239d0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

整个子系统由数据容器以插件的形式运行于客户端，它管理着数据资源，每个dataProvider对应一种数据源，它通过两种方式来获得外部数据。

- 一种是通过查找serviceInfo资源，然后基于其配置，以Dubbo协议（泛化调用方式）或HTTP协议的方式获取。

- 另一种通过本地服务调用的方式获取，接着根据dataStore中的业务代码（source.groovy）来聚合及处理这些数据服务的返回结果，并最终将获取的数据提供给app使用，完成app的取数与渲染，呈现在所有基于店铺平台客户端的应用页面之中。

![插件体系](http://upload-images.jianshu.io/upload_images/1124873-6d20458596683b47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Tag](http://upload-images.jianshu.io/upload_images/1124873-afb929464fcff4ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![管理DataSource](http://upload-images.jianshu.io/upload_images/1124873-b7eecdeb472e5ad8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 开放的设计平台
在核心技术日趋完善的情况下，我们发现生产力依然低下，因为我们要开发的业务需求还是非常多，虽然大家的开发效率已经得到大幅度的提升！

![模板流程](http://upload-images.jianshu.io/upload_images/1124873-330a83241a4162b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是还是会出现乱拳打死老师傅的情况，面对日益增长的个性化需求，我们选择了开放，将店铺核心体系开放出去，接入设计师来帮助我们完成个性化部分！

这其实也是典型的云服务开放的案例，通过设计平台，我们既解决了自身的资源问题，又养活了40多个设计师！

最后店铺业务只需要1-2个人日常维护就可以满足不断增长的业务需求！

# 店铺生态建设
围绕着平台核心技术的打造过程，我们自然逐步完成连接开发者、设计师、第三发开发者、运营人员，他们都可以很好的工作在店铺平台中完成各自的需求。整个店铺业务形成闭环，自然高效的工作在一起！

![店铺生态建设](http://upload-images.jianshu.io/upload_images/1124873-d2910d7110ba5841.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
![](http://upload-images.jianshu.io/upload_images/1124873-98f096df9a20e728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)