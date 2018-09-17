# Android中报找不到SO文件问题

## 问题分析

在API23及以上版本，导入前驱第三方SO库的时候，或许很多人会遇到找不到SO库的问题。去年也有遇到该问题，配合着C++的同事弄了很久才解决掉，一切源于规范。

## 解决方案

问题出现了就需要解决。

## 方案依据

在EFL文件中，EFL头部有一个DT_NEEDED字段，用来记录该动态库所依赖的动态库的绝对路径，但是在Android设备上该字段没有任何意义，因为正常情况下作为APP开发者我们没有权限去控制动态库安装位置(无法确定APP安装后动态库的绝对路径)。所以，DT_NEEDED字段中的内容必须和SO_NAME字段值保持一致。这样系统就会自动在运行社动态的去查找具体的库位置。

事实是，在API 23之前，Android中的动态链接器忽略了DT_NEEDED中的完整路径，并且在查找的时候只使用字段中的基础名词(即最后一个'/'之后的部分)。但是从API 23开始，动态链接器将完全尊徐DT_NEEDE,因此，如果库并不存在与其所指向的位置将导致加载失败。

以上内容参考于：[Invalid DT_NEEDED Entries (Enforced since API 23)](https://android-developers.googleblog.com/2016/06/android-changes-for-ndk-developers.html)

所以遇到这种问题时应该怎么解决呢？