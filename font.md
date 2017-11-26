这篇文章会从字体相关的各个维度去探讨如何在Mobile Web上显示最佳的字体、行高、字重，以及如何实现精确到1px的文字垂直居中。

文章很长，这里直接给出几个核心的结论：

移动端字体设置，全局按如下方式：

font-family: system-ui, -apple-system, BlinkMacSystemFont, Helvetica Neue, Helvetica, sans-serif
解决中文网站在Android下文字垂直居中问题：

<html lang="zh-cmn-Hans">、font-family: -apple-system, BlinkMacSystemFont, sans-serif
Android 7可以通过以上方案解决，但是当前语言是zh且不设置英文字体族会导致Android 5下出现英文fallback到Noto Sans Myanmar的问题，这个字体不但难看而且默认行高惊人，在没有找到兼容方案前请放弃这点优化
行高设置：全局不设置行高（即保持normal），各个字体有自己特有的安全行高，iOS和Android下自带字体的安全行高是1.03～1.18，设置行高1是一个非常危险的操作

字重设置：Android下自带中文思源黑体只有一个字重normal，加一个伪粗体bold，iOS下自带中文苹方有100～600六个字重，所以跨端的中文加粗使用bold或者700，700以下的字重在Android下会直接fallback到normal

PS：**这篇文章的所有探讨限于iOS & Android，并且Android Rom众多，厂商又可以修改字体配置，甚至增删字体，所以大部分测试和结论都针对原生ROM**。
文字的对齐、渲染都和字体有关，所以在讨论任何文字排版优化之前，我们先来认识几个移动端的重要字体：

-apple-system、BlinkMacSystemFont：-apple-system是Apple在iOS 9，OSX 10.11中引入，通过它可以声明使用系统最新的San Francisco字体，并且未来系统更新字体后也可以自动指向新的字体，而BlinkMacSystemFont是Chrome针对-apple-system的类似实现
system-ui：是针对-apple-system和BlinkMacSystemFont的标准实现，Chrome 56+、Safari 10+开始支持，所以类似场景可以这样声明：font-family: system-ui, -apple-system, BlinkMacSystemFont, sans-serif
Noto Sans CJK SC：即思源黑体 - 简体中文，思源黑体是Google和Adobe合作开发的开源字体，是Android 5.0以后的默认中文字体，也是唯一的中文字体，该字体本身包含7种字重，而Android只安装了Regular这一个字重，Chrome开发者工具里显示：Noto Sans CJK SC Regular，因为Android里没有针对该字体设置family name，所以无法直接通过font-family指定，只能作为fallback来使用
Roboto：这是Google专门为Android开发的开源字体，Android 4.0以后的默认英文字体，无法直接通过font-family指定，使用：font-family: system-ui
San Francisco：最早是Apple为watchOS设计的字体，后来在OS X 10.11 El Capitan和iOS 9中取代了Helvetica Neue，成为默认的英文字体，该字体在Chrome开发者工具里查看的话会显示成.SF NS Text（包含6种字重）和.SF NS Display（包含9种字重），字号小于20px时使用.SF NS Text，20px以上时使用.SF NS Display，**可以使用font-family: system-ui, -apple-system, BlinkMacSystemFont来声明**，如果你非要手动声明，字号20px以上时使用：.SFNSDisplay-Black、.SFNSDisplay-Bold、.SFNSDisplay-Heavy、.SFNSDisplay-Light、.SFNSDisplay-Medium、.SFNSDisplay-Regular、.SFNSDisplay-Semibold、.SFNSDisplay-Thin、.SFNSDisplay-Ultralight，字号小于20px时可以用：.SFNSText-Bold、.SFNSText-Heavy、.SFNSText-Light、.SFNSText-Medium、.SFNSText-Regular、.SFNSText-Semibold
PingFang SC：即苹方 - 简体中文，这是Apple在iOS 9时专门设计的中文字体，和英文字体San Francisco配合非常协调，对应的font-family名字是：PingFang SC，包含6种字重
从以上组合进一步总结：

当前语言是zh且不声明英文字体族时，Android Chrome对英文也会fallback到Noto Sans CJK SC，所以声明英文字体族是有必要的，思源黑体+Roboto的搭配也还算和谐，但下文讲文字垂直居中时会提到这种设置导致的问题
当不声明英文字体族时，iOS Safari会将英文fallback到Helvetica，甚至不是上一代的系统默认字体Helvetica Neue，所以声明英文字体族是有必要的，可以保证使用最佳的英文字体San Francisco，中英混排时中文会fallback到苹方，苹方和San Francisco是同时随iOS 9发布的，2者配合非常和谐，也不需要担心2者混排会存在对齐问题
移动端世界没有中文衬线字体(font-family: serif)，所以不要在中英混排的场景下设置衬线字体，否则衬线、非衬线在一起显示很不协调
如何设置line-height
对于行高的设置建议直接阅读@大貘 翻译的深入了解CSS字体度量，行高和vertical-align，总结几点：

字体的font-size不等于最终显示的大小，取决于字体设计师的定义，以Catamaran字体为例，当我们设置100px的字号时，大写字母的高度（Capital Height）是68px，小写字母的高度（X-Hegiht）是49px，整体的高度是164px（Ascender 1100 + Descender 540），要了解任意字体的相关属性可以使用(fontforge)[https://fontforge.github.io/en-US/]
我们日常使用的CSS单位em对应font-size本身，ex对应小写字母的高度（中文字体对应的也是该字体下英文小写字母的高度）
正因为第一点，为了不截掉文字，**最小行高的设置需要依据字体来**，以Arial为例，设置line-height: 1时就会造成文字下面被截掉的问题：

你可能并没有注意到过这个问题，用Android Chrome打开你的页面，仔细观察页面上需要文字垂直居中的元素，你会发现文字要么偏上要么偏下，不管是Google、Facebook、Twitter等国际一线公司（Apple没有问题，原因是通过Web Font使用了PingFang SC），还是国内的BAT都一样（包括专门搞内容的知乎、简书等）

这个问题无论你是用flex、行高、表格+vertical-align等都无法解决，因为文字在content-area内部渲染的时候已经偏上了，CSS的各种居中方案都是针对整个content-area内容。

结合前面的分析，移动端最佳的字体设置如下：

结论
font-family: system-ui, -apple-system, BlinkMacSystemFont, Helvetica Neue, Helvetica, sans-serif;
system-ui, -apple-system, BlinkMacSystemFont：这3个关键字都用来选择当前系统的最佳英文字体
Helvetica Neue：用于让不支持第一条且没有San Francisco字体的iOS 8以下环境使用最佳的英文字体Helvetica Neue
Helvetica：针对不支持第一条的Chrome 56以下端，设置成Helvetica的原因是Android的字体配置文件里会将Helvetica字体alias到sans-serif，而sans-serif对应当前版本下的最佳英文字体（Android 5.0以上就是Roboto）

<alias name="helvetica" to="sans-serif" />
sans-serif：声明使用无衬线字体，因为"移动端没有衬线中文字体"，防止中英混排时出现衬线+非衬线的问题