> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.jianshu.com/p/b87ca8615c9c

> 代码不只是用来运行的，更是用来阅读的

## 编码规范

代码规范看似只是简单的一个编码的规范，对代码格式、变量命名、注释格式等做一个统一规范，看似有点强迫你改变编码风格的味道。但是当你和许多团队协同开发过许多项目，写过许多代码之后再回头来看的话，你就会发现它真的就是个规范，啊不，真的是个好东西。有没有看过国庆阅兵上的方阵，当所有军人着装统一，排列整齐，一起从长安街走过得时候，会不会被他们带来的视觉冲击所震撼。编码规范就如其中的军规，当大量的代码都具有统一性整齐性时，带给每个参与的人的视觉效果也是极具冲击性的。当然除了视觉效果，愉悦了心情之外，这种统一性，为每个参与编码的人员统一了风格，彼此之间互相阅读代码的效率就高了，团队间的协作效率也就高了。常见的比较受认可的编码规范，基本都是大厂的（废话，没点头衔在高傲的程序猿前谁会服谁），例如：Google、Sun、阿里巴巴等大厂都有发布自己的编码规范，这些可都是集万千大神的工程实践总结，这些总结帮助行业人员**提高了开发质量和效率，大大降低了代码的维护成本**。

## 阿里巴巴 JAVA 编码规范

好多大厂都发布了编码规范，为何选阿里巴巴的呢？第一，阿里给的规范更加符合国人的习惯，毕竟里面绝大部分都是国人；第二就是阿里的规范文档都是中文的，方便阅读，也降低了一些翻译的歧义性等等。（真实理由其实就一个，我英文不好）有用过谷歌规范的再去用阿里的就会深有体会了，谷歌的规范毕竟比较符合西方人习惯，很多注释上的规范很蛋疼的。

阿里巴巴的 JAVA 编程规范，至今为止已更迭了多个版本，2018 年 6 月 6 日，《阿里巴巴 Java 开发手册（详尽版）》v1.4 正式在 GitHub 上发布，这是史上内容最全、修正最为彻底的一个版本，并且增加了单元测试规约内容，这也是阿里官方对外发布的最后一个 PDF 版本，值得收藏。

《阿里巴巴 Java 开发手册》是阿里内部 Java 工程师所遵循的开发规范，涵盖编程规约、单元测试规约、异常日志规约、MySQL 规约、工程规约、安全规约等，这是近万名阿里 Java 技术精英的经验总结，并经历了多次大规模一线实战检验及完善。这是阿里回馈给 Java 社区的一份礼物，希望能够帮助企业开发团队在 Java 开发上更高效、容错、有协作性，提高代码质量，降低项目维护成本。

这里是[阿里巴巴 Java 编码规范的 GitHub 传送门](https://github.com/alibaba/p3c)。里面提供了多种代码规范检测方式：

*   IntelliJ IDEA 插件集成方式
*   Eclipse 插件集成方式
*   pmd 工具集成方式

对应的 IDE 插件可以看一下这篇文章《[阿里巴巴 Java 开发规约插件 p3c 详细教程及使用感受](http://www.cnblogs.com/han-1034683568/p/7682594.html)》

为了和 GitLab 能够配合，进行提交时自动化得代码检查，这边选择了第三种 pmd 工具集成方式。

## 打成 PMD 工具 Jar 包

由于阿里的 GitHub 中 p3c/p3c-pmd 提供的只是规则代码，主体 PMD 代码是以依赖方式引入，要把它制作成一个可独立运行的工具包，需要将所有依赖的包都封装到一起，打成一个胖 Jar 包，才能够在本地独立得运行。

1.  从 GitHub 中把 p3c 的代码库下载下来（可用 git clone 也可以直接在 GitHub 中下载 zip 包）
2.  安装 Gradle
3.  进入到 p3c-pmd 目录中，初始化 Gradle 项目

```
# gradle init

```

1.  编辑 build.gradle，加入 jar 块（最后一块代码块）

```
apply plugin: 'java'
apply plugin: 'maven'

group = 'com.alibaba.p3c'
version = '1.3.6'

description = """p3c-pmd"""

sourceCompatibility = 1.7
targetCompatibility = 1.7
tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}

configurations.all {
}

repositories {
     maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
     maven { url "http://repo.maven.apache.org/maven2" }
}
dependencies {
    compile group: 'net.sourceforge.pmd', name: 'pmd-java', version:'5.5.2'
    compile group: 'net.sourceforge.pmd', name: 'pmd-vm', version:'5.5.2'
    testCompile group: 'net.sourceforge.pmd', name: 'pmd-test', version:'5.5.2'
}
jar {
    from {
        // 添加依懒到打包文件
        configurations.runtime.collect{zipTree(it)}
    }
}

```

1.  开始构建 Gradle 项目

Windows 下：

```
cd /path/to/p3c-pmd/
gradlew.bat build

```

Linux 下：

```
cd /path/to/p3c-pmd/
./gradlew build

```

1.  构建完成后在 build/libs / 中会生成 p3c-pmd-1.3.6.jar 包，到此就获取到了我们需要的胖 Jar 包了
2.  为啥会有第 7 步，直接拿这个胖 Jar 去检查文件的话，会出现如下问题：

```
java -cp p3c-pmd-1.3.6.jar net.sourceforge.pmd.PMD -d test.java -R rulesets/java/ali-comment.xml

```

执行后报错：

```
Exception in thread "main" java.lang.NullPointerException
        at net.sourceforge.pmd.cli.PMDParameters.getLanguage(PMDParameters.java:223)
        at net.sourceforge.pmd.cli.PMDParameters.transformParametersIntoConfiguration(PMDParameters.java:151)
        at net.sourceforge.pmd.PMD.run(PMD.java:490)
        at net.sourceforge.pmd.cli.PMDCommandLineInterface.run(PMDCommandLineInterface.java:167)
        at net.sourceforge.pmd.PMD.main(PMD.java:477)

```

为啥空指针异常了，去查看 PMD 源码，发现 PMD 获取不到资源文件中的语言值。而这个资源文件在 META-INFO 中。上面的构建方式没有去解决 Jar 包中 META-INFO 文件下的文件合并问题。每个依赖的 jar 包在合并时都是复制进来，这就导致了原先的 PMD 包中有 net.sourceforge.pmd.cpd.Language 和 net.sourceforge.pmd.lang.Language 这两个文件，而 p3c-pmd 中也有这两个文件，冲突了。

我们要做得就是将包解压出来，然后确保这两个文件都只有一份在 META-INFO 文件夹中，并且保证:

net.sourceforge.pmd.cpd.Language 文件的值为

```
net.sourceforge.pmd.cpd.JavaLanguage

```

net.sourceforge.pmd.lang.Language 文件的值为

```
net.sourceforge.pmd.lang.java.JavaLanguageModule

```

最后重新打成 Jar 包就好了。至此，生成的 jar 包才是正确可用的编码规范检查工具。
命令：jar cvfM p3c-pmd-1.3.6.jar -C p3c-pmd-1.3.6/ .
这里，很多人肯定用多了 IDE，都已经忘记 java 中 jar 包基本的操作了，[这里附上 Jar 包基本的操作传送门](https://blog.csdn.net/qq_22073849/article/details/77148847)。
还有，[Java 打 Jar 包的几种方式](https://www.cnblogs.com/mq0036/p/8566427.html)。

## 检查源码

接下来就可以用生成的 jar 进行代码检查了

```
java -cp p3c-pmd-1.3.6.jar net.sourceforge.pmd.PMD -d E:\CodeRepos\data-server-test\data-server-core\src\main\java\com  -R rulesets/java/ali-comment.xml
E:\CodeRepos\data-server-test\data-server-core\src\main\java\com\jiniutech\common\BeanConvert.java:6:   【BeanConvert】 注释缺少@author信息
E:\CodeRepos\data-server-test\data-server-core\src\main\java\com\jiniutech\common\BeanConvert.java:7:   接口方法【getBean】必须使用javadoc注释

```

参数解释：

*   -d 源码目录，多个文件或者目录以, 号分开
*   -R 指定规则，多个规则以, 号分开。阿里规则路径在包中 rulesets/java/ali-*.xml
*   -f 报告格式，text html xml 等。

附:[PMD 工具官方网站](https://pmd.github.io/)

## 集成到 GitLab 的 hooks 中

hook 机制在很多系统中都有，hook 机制使得 GitLab 能在特定的重要动作发生前 / 时 / 后触发自定义的脚本。GitLab 在每次项目 init 时，就会在每一个项目里创建一个 hooks 文件夹的软链接，指向一个特定文件夹。所以在 hooks 里的操作都是全局操作，是面向所有项目的。

```
[root@localhost data-server.git]# ll -la 
total 32
lrwxrwxrwx.  1 polkitd root   47 Oct 25 02:43 hooks -> /opt/gitlab/embedded/service/gitlab-shell/hooks

```

这里是 [GitLab Hooks 的官方文档](https://git-scm.com/docs/githooks)。

由于我们想要达到的效果是每次提交前进行代码检查，因此要用到 pre-receive 文件，首先将 p3c-pmd-1.3.6.jar 包复制到 hooks 中，然后在 hooks 中创建 pre-receive 文件，内容如下：

```
#!/bin/sh
#

REJECT=0

while read oldrev newrev refname; do
    if [ "$oldrev" = "0000000000000000000000000000000000000000" ];then
        oldrev="${newrev}^"
    fi

    files=`git diff --name-only ${oldrev} ${newrev}  | grep -e "\.java$"`

    if [ -n "$files" ]; then
        TEMPDIR="tmp"
        for file in ${files}; do
            mkdir -p "${TEMPDIR}/`dirname ${file}`" >/dev/null
            git show $newrev:$file > ${TEMPDIR}/${file}
        done;

        files_to_check=`find $TEMPDIR -name '*.java'`

        /home/jdk1.8.0_191/bin/java -cp hooks/p3c-pmd-1.3.6.jar net.sourceforge.pmd.PMD -d ${files_to_check} -R rulesets/java/ali-comment.xml,rulesets/java/ali-concurrent.xml,rulesets/java/ali-constant.xml,rulesets/java/ali-exception.xml,rulesets/java/ali-flowcontrol.xml,rulesets/java/ali-naming.xml,rulesets/java/ali-oop.xml,rulesets/java/ali-orm.xml,rulesets/java/ali-other.xml,rulesets/java/ali-set.xml -f text
        REJECT=$?

        rm -rf $TEMPDIR
    fi
done

exit $REJECT

```

要注意的是 pre-receive 文件必须没有任何后缀，且为可执行文件 (+X)。

这个脚本在每次提交前检查所有提交的后缀名. java 的文件，然后用得到的 p3c-pmd-1.3.6.jar 包对这些文件进行代码检查，然后返回结果。如果存在不规范的代码则返回给提交者错误信息，如下图

![](https://upload-images.jianshu.io/upload_images/6817147-c6c6d5ff63458d8d.png)

这样强制进行代码检查就完成了。

附：<a href="" target="_blank">Docker 中安装 Java（待补全）</a>

