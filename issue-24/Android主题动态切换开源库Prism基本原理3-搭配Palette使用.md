#Android主题动态切换开源库Prism基本原理3-搭配Palette使用

> * 原文链接 : [Prism Fundamentals Part 3](https://blog.stylingandroid.com/prism-fundamentals-part-3/)
* 原文作者 : [Mark Allison](https://blog.stylingandroid.com/)
* 译文出自 : [开发技术前线 www.devtf.cn。未经允许，不得转载!](www.devtf.cn)
* 译者 : [Desmond1121](https://github.com/desmond1121)  

---

**译者注：在阅读本章之前你应该确定你已经了解[Android Palette](https://developer.android.com/reference/android/support/v7/graphics/Palette.html)。**

关于Palette，这里有一篇参考文章写得不错:[Extracting Colors to a Palette with Android Lollipop](https://www.bignerdranch.com/blog/extracting-colors-to-a-palette-with-android-lollipop/) （我后续会翻译这篇文章）

---

**重要提示：Prism源码目前停止更新了（你可以在[Prism-Github](https://github.com/StylingAndroid/Prism) 描述文件中看到）。不过我还是决定写出这一系列的文章来介绍Prism现在的版本，因为它很可能还有用。**

Previously we say how ViewPagerTrigger can easily be hooked up to Prism in order to drive the colouring of our UI quite easily. As well as ViewPager integration, Prism v1.0.0. also has triggering from the Palette support library in an optional module named prism-palette and in this article we’ll look at how we connect this all up.

在前面一章中我们介绍了怎么使用Prism和ViewPager结合起来。在Prism 1.0.0版本中同样也有和Palette相结合的库，我们现在就来看看怎么去使用它。

First we need to add prism-palette as a dependency to our project:

首先我们向`build.gradle`中添加相关的依赖：

	apply plugin: 'com.android.application'

	android {
	    compileSdkVersion 22
	    buildToolsVersion "23.0.0 rc3"
	
	    defaultConfig {
	        applicationId "com.stylingandroid.prism.sample.palette"
	        minSdkVersion 7
	        targetSdkVersion 22
	        versionCode 1
	        versionName "1.0"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	}
	
	dependencies {
	    compile 'com.android.support:appcompat-v7:22.2.0'
	    compile 'com.android.support:design:22.2.0'
	    compile 'com.stylingandroid.prism:prism:1.0.0'
	    compile 'com.stylingandroid.prism:prism-palette:1.0.0'
	}
	
PaletteTrigger itself is really simple – we just need to create a new PaletteTrigger instance and add it to the Prism builder:

`PaletteTrigger`本身的使用是非常简单的，我们只要新建一个`PaletteTrigger`实例然后将它加入到`Prism.Builder`就好了，代码如下：

	paletteTrigger = new PaletteTrigger();
	prism = Prism.Builder.newInstance()
        .add(paletteTrigger)
        /* 添加其他组件到Prism */
        .build();
		
Once this is done we can trigger colour changes by calling setBitmap(Bitmap bitmap) on the PaletteTrigger. This will create a new Palette instance for the supplied Bitmap and then trigger Prism once the Palette extraction is complete.

当加入`PaletteTrigger`后我们就可以通过调用`PaletteTrigger.setBitmap(Bitmap bitmap)`来提取目标Bitmap的特征颜色。这个函数会新建一个`Palette`实例来处理支持的图片格式，并在处理结束后触发Prism颜色变换。

However, because of the way that Palette works we need to get a little creative in how we wire up our UI components to get them appropriately coloured. TO under stand this, let’s have a quick recap on how Palette works.

由于`Palette`的特殊机制我们需要用一点手段将UI组件和`Prism`结合起来，从而它们能够被正常染色。首先让我们回顾一下`Palette`是怎么工作的：

Palette returns up to six separate colour Swatches for vibrant, dark vibrant, light vibrant, muted, dark muted, and light muted colours which have been extracted from the image. For each of these Swatches we can retired one of three colour values: The raw colour, a colour value appropriate for any title text which has a background of the raw colour, and a colour value appropriate for any body text which has a background of the raw colour. So there are up to eighteen different colour variants that we can retrieve from the Palette.

`Palette`从图片中抽取出6种不同的颜色样本`Swatch`：vibrant(鲜艳色)、dark vibrant(鲜艳色中的暗色)、light vibrant(鲜艳色中的亮色)、muted(柔和色)、dark muted(柔和色中的暗色)、light muted(柔和色中的亮色)。
你可以通过颜色样本`Swatch`获取三个颜色值：

- 抽取出的初始颜色(`Swatch.getRgb()`或`Swatch.getHsl()`)
- 与抽取出颜色搭配的标题字体色(`Swatch.getTitleTextColor()`)
- 与抽取出颜色搭配的正文字体色(`Swatch.getBodyTextColor()`)

所以我们一共能从`Palette`中获取到18个颜色值。

PrismTrigger provides a number of factory methods which return the various Swatches in the form of Filters, and each of these takes a modifier to determine whether the raw, title colour, or body colour will be used. So we are essentially leveraging the Filter mechanism to extract the appropriate colour for each of the UI components that we wire up through Prism.

`PaletteTrigger`提供了一系列工厂方法来获取`Swatch`，并返回一个`Filter`。这个`Filter`会告诉`Setter`要给View设置的颜色是初始颜色、标题字体色、正文字体色中的哪一个。所以我们使用`Filter`来帮助`Prism`链接UI组件是非常必要的。

So, to get a dark vibrant, title text colour we simply chain up the appropriate factory methods to create the filter that we require:

要获取与图片中dark vibrant色相搭配的正文色，只需要下面这一行代码就可以实现：

	Filter darkVibrantTitle = paletteTrigger.getDarkVibrantFilter(paletteTrigger.getTextFilter()); 

If no filter is set then the vibrant raw colour value will be used, however it is recommended that you wire up what you require. Currently if a particular swatch is not found by Palette then transparency will be applied (effectively hiding the UI component being coloured). This is not ideal and there will be improvements to this in a future release.

如果不为View设置`Filter`的话，那么会默认使用初始颜色（不过我还是建议你根据使用目的进行设置）。目前的版本中如果对应的颜色种类在`Palette`中没有抽取到的话会使用透明色（即绑定的UI组件会在屏幕上暂时消失），这点还不是很令人满意，我会在后续的开发中进行改进。

So the entirety of our PaletteTrigger is wired up thus:

那么我们已经清楚了`PaletteTrigger`是怎么使用的了，为了有个更具体的理解，下面这段`onCreate()`中的代码片段中展示了整个使用过程：

	View vibrant = findViewById(R.id.swatch_vibrant);
	View vibrantLight = findViewById(R.id.swatch_vibrant_light);
	View vibrantDark = findViewById(R.id.swatch_vibrant_dark);
	View muted = findViewById(R.id.swatch_muted);
	View mutedLight = findViewById(R.id.swatch_muted_light);
	View mutedDark = findViewById(R.id.swatch_muted_dark);
	
	titleText = (TextView) findViewById(R.id.title);
	bodyText = (TextView) findViewById(R.id.body);

	paletteTrigger = new PaletteTrigger();
	prism = Prism.Builder.newInstance()
	    .add(paletteTrigger)
	    .background(vibrant, paletteTrigger.getVibrantFilter(paletteTrigger.getColour()))
	    .background(vibrantLight, paletteTrigger.getLightVibrantFilter(paletteTrigger.getColour()))
	    .background(vibrantDark, paletteTrigger.getDarkMutedFilter(paletteTrigger.getColour()))
	    .background(muted, paletteTrigger.getMutedFilter(paletteTrigger.getColour()))
	    .background(mutedLight, paletteTrigger.getLightMutedFilter(paletteTrigger.getColour()))
	    .background(mutedDark, paletteTrigger.getDarkMutedFilter(paletteTrigger.getColour()))
	    .background(titleText, paletteTrigger.getVibrantFilter(paletteTrigger.getColour()))
	    .text(titleText, paletteTrigger.getVibrantFilter(paletteTrigger.getTitleTextColour()))
	    .background(bodyText, paletteTrigger.getLightMutedFilter(paletteTrigger.getColour()))
	    .text(bodyText, paletteTrigger.getLightMutedFilter(paletteTrigger.getBodyTextColour()))
	    .add(this)
	    .build();
		
This wires up 6 View objects to take each of the vibrant, dark vibrant, light vibrant, muted, dark muted, and light muted swatches, plus two TextViews which will use the vibrant, title text, and light muted, body text Filters respectively.

这段代码将六个`View`组件分别赋予了`Palette`解析后的六种初始颜色，并分别为两个`TextView`的文本赋予了 *鲜艳色搭配的标题文本色*与 *亮柔和色搭配的正文文本色*。

You may also notice that we’re registering the Activity itself as a Setter. This is to get a callback when the Palette extraction is complete as this can be slow on larger images. This allows us to wait until the Palette extraction is complete before we actually update the image in the ImageView within our layout. This just provides a slightly nicer UX as the image updates at the same time as the UI colours get updated by Prism.

你可能注意到了我们将`Acitivty`实现成了`Setter`（通过`add(this)`这一行看出）。这是为了实现在`Palette`处理好图片后的回调函数（即`setColour(int colour)`)中设置`ImageView`的图片，由于`Palette`在解析大图片的时候需要花费一些时间，这么做可以让解析颜色与显示图片的结果同时显示到屏幕上，用起来更加舒服。

So let’s see this in action (regular readers of Styling Android may recognise Betty who appears occasionally):

我们来看看运行起来是什么效果吧：

![Demo](http://img.blog.csdn.net/20150817155549561)

So we’re not actually wiring the UI of the app in this case, just giving some examples of how you can extract the various swatch variants and use them. But if you’re read the previous parts of this series it should be fairly straightforward to adapt this.

你也可以看到，我们实际上没有将其他UI组件（`Toolbar`等）加入到这个Demo中，只是为获取颜色样本做了示例。如果你已经阅读并理解了前面几篇文章的话，你能够很容易地掌握如何将`PaletteTrigger`结合到主题颜色变换的应用中。

That concludes our look at the basics, and we’ll finish with Prism for now. However, we’ll re-visit if and when active Prism development resumes.

那么基础的部分就介绍完毕了，现在我们会把Prism的开发告一段落。当Prism恢复开发之后，我会再写更多的东西来介绍它的更多应用。

这里面所有的例子都会在[源码Github](https://github.com/StylingAndroid/Prism)中的[示例代码](https://github.com/StylingAndroid/Prism/tree/master/sample-palette) 中看到。

