
# Data model for a social reader application 


在线阅读和共享新闻及其他文本的应用程序一直在变化。从Slashdot到StumbleUpon再到Planet，似乎还没有人知道它是否正确。好消息是，现在似乎每个
人都在写某种社交阅读器，因此对于许多人来说，它应该是一个熟悉的领域，对于示例代码而言是如此。谁知道呢？也许这次我们会做对的。

我想到的社交阅读器应用程序介于已故的出色Google阅读器和共享网站（如Digg或Reddit）之间。像Google阅读器一样，用户将能够阅读故事的文本，并且
应用程序将跟踪他们所看到的内容。像Reddit一样，他们可以上下投票故事，而投票将影响其他用户观看故事的顺序。


数据非常基础。

有成员，故事和评分。成员具有URL和分数。

一个故事有一个ID和一个文本。棘手的是，同一故事可能具有多个URL。 （通常是同一URL的“ http”和“ https”版本，或者带有和不带有“ www”或“ 
index.html”的相同URL。）但是，故事恰好具有一个首选URL。

对于喜欢故事的用户，评分为正面；对于不喜欢故事的用户，评分为负面；对于表示用户看过该故事且不应再次显示该故事，但不对其进行上下评级的，
则评分为0。

我们可以将评级视为来自可以是用户的“主题”，外部推荐者（例如RSS feed或社交网站）或内部软件组件（例如主页构建器）。任何主题都可以对任何“对象”
（通常是故事）进行投票。

## Queries for Social Reading

他们总是告诉您在设计CQL数据模型时要考虑查询，那么此应用程序需要哪种查询？因为它是一个简单的应用程序，所以只有几个主要的应用程序。

* 获取最新故事。
* 自给定日期以来，给定用户评分最高的故事。这将使我们为每个想要一个用户的用户建立一个最近推荐的页面。
* 评分最高的故事尚未由给定用户评分。这是为了向用户显示“您还没有读过的新故事”，并为注销的用户建立主页。主页可以仅表示为另一个用户，当故事
出现一定时间后，该故事的评分为0。

对于更新，我主要需要能够撰写新的评分和故事。这是数据模型的第一个版本。


    CREATE TABLE rating_by_subject (
        subject VARCHAR,
        object VARCHAR,
        score INT,
        date TIMESTAMP,
        PRIMARY KEY (subject, date)
    );
    
    CREATE TABLE rating_by_object (
        subject VARCHAR,
        object VARCHAR,
        score INT,
        date TIMESTAMP,
            PRIMARY KEY (object)
        );

这三个版本可帮助我们快速查找给定等级是否存在，以及获得给定主题和物体的所有等级。通常，主题是用户，而对象是故事，但系统中其他实体之间也可能
存在评级。例如，用户可能能够相互评价，或者RSS feed可以对它发布的所有故事和链接进行评价。故事非常简单，主要是内容，还有几kb的HTML，带有故
事URL的主键。故事的大多数属性都可以在HTML中，只有一个“类型”可以跟踪此URL是否代表另一个首选URL上的故事的副本。

    
    CREATE TABLE story (
       url VARCHAR,
       date VARCHAR,
       type VARCHAR,
       content VARCHAR,
       PRIMARY KEY(url, date)
    );
    Copy to clipboard

该日期用作聚类键。

最后一张表涵盖成员。成员可以是用户，RSS提要或任何其他推荐来源。为了避免用额外的列弄乱这个示例，并避免使用密码弄乱用户的例程，我们将使
用OAuth或Mozilla Persona之类的第三方登录服务。用户可以使用现有的网络或电子邮件帐户登录，而我们不必编写密码恢复功能。

    
    CREATE TABLE member (
       url VARCHAR,
       score INT,
       active TIMESTAMP,
       success TIMESTAMP,
       PRIMARY KEY(url)
    );
现在开发此应用程序的主要问题是，某些查询在Apache Cassandra空间中很难执行。在SQL中这很容易实现，因为我们的查询需要过滤掉给定用户已经看到
的所有故事，因此在CQL中是禁止的。

但是您不能真正使Cassandra像RDBMS一样工作。解决方案是让Cassandra做好自己最擅长的事情，并将其余工作放在客户端上。

与其更改数据库，不如仅通过发送最近看不见的故事的列表来节省带宽，而是对其进行更改。对于社交阅读器应用程序，经济很重要。查询后处理（这种查询
处理可以使您在RDBMS支持的学校项目中获得F）不仅对社交读者有用。免费。因为您可以在用户付费的客户端上进行此操作，而不是在账单所在的数据中心
内进行。

因此，让我们重新考虑未分级故事的查询。我们不会只一步一步地进行整个查询，而只是将新故事的URL发送给客户端，然后让客户端请求自该新故事最早的时
间戳以来该用户的观看列表。客户将过滤掉已经看过的故事。

我们将在每台客户端设备上保留一个“不要给我发送任何比此时间更早的”时间戳，以使每个用户的不同设备保持同步。

第一个查询将是：

    
    SELECT url FROM story WHERE date > ? LIMIT ?
    

接下来，我们使用第一个查询中返回的最早故事的时间戳来获取要过滤的URL列表。

    
    SELECT object FROM rating_by_subject
          WHERE subject = ?
          AND date > ?
          LIMIT ?

这在MSIE 6天之内是行不通的，但是今天的浏览器具有LocalStorage，可以让我们不经常更新故事列表。我们可以调整返回的故事数，以平衡网络流量。

最后，对于构建提要和主页，查询中的主要标准是故事的新鲜度。因此，我们可以使用rating_by_date表。

    
    SELECT object FROM rating_by_date
          WHERE date > ?
          LIMIT ?
  
NoSQL之所以起作用，部分原因在于现代的客户端层很棒-在图形，处理能力和localStorage之间，您需要在服务器上执行的所有代码都是用于实现安全性或
数据一致性的代码。也许使用Scylla我们将永远不会制作一个真正的社交阅读器应用程序，但是该设计值得尝试