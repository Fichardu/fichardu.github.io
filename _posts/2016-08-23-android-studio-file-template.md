---
layout: post
title:  "Android Studio 自定义文件模版"
date:   2016-08-23
categories: Android
---

Android Studio 可以一次生成多个模版文件，如 BlankFragment，会产生一个 Fragment，一个 layout 资源文件和一个 string 资源文件。使用文件模版可以显著减少代码编写工作，比如现在 Android 开发使用的 MVP 架构，每个模块都是相似的文件结构，像 Presenter、View、Repo等。

Android Studio 模版文件位置在`/Applications/Android Studio.app/Contents/plugins/android/lib/templates/other/`目录下，对应文件夹下是各个模版的配置文件，主要包括：

- template.xml
- recipe.xml.ftl
- globals.xml.ftl
- root/
	- src
	- res

下面以一种 MVP 结构为例说明一下模版文件的具体写法：

## template.xml

这个文件包含了模版的配置参数，如 minApi、minBuildApi、name、description，`<category/>` 标签指定模版在菜单中显示的分类名，`<parameter/>` 指定生成模版时需要用户填写的参数；

	<template format="4"
		revision="1"
		name="MVP Template"
		description="Creates a new MVP classes - Presenter, Contract and RepoImpl">

		<category value="Other"/>

		<parameter id="className"
			name="ClassName"
			type="string"
			constraints="class|unique|nonempty"
			default="Mvp"
			help="Mvp Class prefix name"/>

		<globals file="globals.xml.ftl" />
	  	<execute file="recipe.xml.ftl" />

	</template>

## globals.xml.ftl

一些生成模版时使用的全局变量，这个文件是可选的；

	<?xml version="1.0"?>
	<globals>
	 <global id="resOut" value="${resDir}" />
	 <global id="srcOut" value="${srcDir}/${slashedPackageName(packageName)}" />
	</globals>

## recipe.xml.ftl

该文件定义了模版文件的生成规则，包括拷贝文件、创建文件等，通过具体的标签来控制；

- <copy /> 拷贝文件
- <instantiate /> 创建文件
- <open /> 打开文件


		<?xml version="1.0"?>
		<recipe>
			<instantiate from="src/app_package/Contract.java.ftl"
		                   to="${escapeXmlAttribute(srcOut)}/${className}Contract.java" />
			<instantiate from="src/app_package/RepoImpl.java.ftl"
		                   to="${escapeXmlAttribute(srcOut)}/${className}RepoImpl.java" />
			<instantiate from="src/app_package/Presenter.java.ftl"
		                   to="${escapeXmlAttribute(srcOut)}/${className}Presenter.java" />
			<open file="${srcOut}/${className}Presenter.java"/>
		</recipe>

## root

root 目录下是每个模版文件的具体内容，src 和 res 分别放在对应的文件夹下；

- root
	- res
		- layout
		- values
	- src
		- app_package
			- Contract.java.ftl
			- Presenter.java.ftl
			- RepoImpl.java.ftl


Contract.java.ftl

```
package ${packageName};

import com.example.BasePresenter;
import com.example.BaseRepo;
import com.example.BaseView;

interface ${className}Contract{

	interface Presenter extends BasePresenter {

	}

	interface View extends BaseView<Presenter> {

	}

	interface Repo extends BaseRepo {

	}

}
```

Presenter.java.ftl

```
package ${packageName};

public class ${className}Presenter implements ${className}Contract.Presenter {

  private ${className}Contract.View mView;
  private ${className}Contract.Repo mRepo;

  public ${className}Presenter(${className}Contract.View view, ${className}Contract.Repo repo) {
    mView = view;
    mRepo = repo;
  }

  @Override
  public void detachFromView() {
    mView = null;
    mRepo.cancel();
  }
}
```

RepoImpl.java.ftl

```
package ${packageName};

import com.souche.fengche.crm.BaseRepoImpl;

public class ${className}RepoImpl extends BaseRepoImpl implements ${className}Contract.Repo {

  public ${className}RepoImpl() {

  }

}
```		

上面的文件保存后重启 Android Studio，就可以在 File －> New -> Other -> Mvp Template 下点击生成模版。

# Reference

[How to create a group of File Templates in Android Studio – Part 3](https://riggaroo.co.za/custom-file-template-group-android-studiointellij/)

[File Templates in Android Studio](https://coderwall.com/p/fsxvyw/file-templates-in-android-studio)
