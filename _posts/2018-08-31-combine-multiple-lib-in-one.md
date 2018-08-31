---
layout: post
title: "将多个lib库组合成一个lib库"
date: 2018-08-31
---

###方法一
1. 创建一个空的Win32静态库项目；
2. 在该项目上新建一个dummy.c文件，该文件可以为空，它存在的目的是为了让该项目能正常的进行编译；
3. 选择该项目，点击鼠标右键，在弹出的右键菜单中选择“属性”，然后在弹出的窗口中选择“配置属性->管理员->常规”，在“附加依赖项中”填入需要合并的静态库。这些静态库可以是相对路径，例如：

		"..\aproject\Debug\aproject.lib"
		"..\bproject\Debug\bproject.lib"


###方法二：
Visual Studio 2008提供了一个工具，叫做lib.exe，这个工具保存在C:\Program Files\Microsoft Visual Studio 9.0\VC\bin\lib.exe中。使用这个工具，用户可以：

* To add objects to a library, specify the file name for the existing library and the filenames for the new objects.
* To combine libraries, specify the library file names. You can add objects and combine libraries with a single LIB command.
* To replace a library member with a new object, specify the library containing the member object to be replaced and the file name for the new object (or the library that contains it). When an object that has the same name exists in more than one input file, LIB puts the last object specified in the LIB command into the output library. When you replace a library member, be sure to specify the new object or library after the library that contains the old object.
* To delete a member from a library, use the /REMOVE option. LIB processes any specifications of /REMOVE after combining all input objects, regardless of command-line order.

所以我们也可以使用如下方法将多个lib合并成一个lib库：

	"C:\Program Files\Microsoft Visual Studio 9.0\VC\bin\lib.exe" /OUT:libabprj.lib aprj.lib bprj.lib
	
