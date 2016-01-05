```
/*
|----------------------------------------
| Hack the Core of PHP
| @weiChen translate, <farwish@live.com>
| @MIT License
*/
```

  http://php.net/manual/zh/internals2.ze1.zendapi.php

  简单的PHP并不总是够用，尽管这种情况对大多数使用者很罕见，专业的应用程序会很快把PHP带到它的性能瓶颈，在速度上或在功能上。


### 存在可能性的扩展

  PHP主要在三个地方可被扩展：外部模块，内置模块，Zend引擎。
  
外部模块：
  外部模块可以在脚本运行时用 dl() 函数加载。这个函数通过在磁盘上加载一个共享对象使模块功能对绑定的脚本可用。
这种方法有好处也有坏处：
  
  ```
   --------------------------------------------------------------------------
  | 好处                          | 坏处                                     |
  |--------------------------------------------------------------------------|
  | 外部模块不需要重新编译PHP     | 共享对象在脚本执行时每次都要加载，非常慢 |
  |--------------------------------------------------------------------------|
  | 通过外包某些功能保持PHP的大小 | 外部的额外文件使磁盘杂乱                 |
  |--------------------------------------------------------------------------|
  ```
  
  每个使用外部模块功能的脚本需要特别包含一个 dl() 的调用，否则需要修改 php.ini 的扩展标签(不总是合适的解决办法)。
  
  总结：外部模块非常适合第三方产品，使用很少的对PHP的小补充或仅仅是测试目的。
为了快速开发额外的功能，外部模块提供了最好的结果。针对频繁的使用、更广泛的执行，和复杂的代码，弊大于利。
第三方可能会考虑使用 php.ini 中的扩展标签来为PHP创建额外的外部模块。这些外部模块完全与主软件包分开，这在商业环境中是非常好用的特性。
商业分销商能简单的运输磁盘或存档仅仅包含他们的附加模块，而不需要创建不允许其他模块绑定它们的固定和坚实的PHP二进制文件。
  
内置模块：
  内置模块直接编译进PHP并被每一个PHP进程携带；它们的功能对每一个正在运行的脚本来说都是即刻可用的。
相比外部模块，内置模块有好处和坏处：

  ```
   -------------------------------------------------------------------------------------------
  | 好处                                                  | 坏处                              |
  |-------------------------------------------------------------------------------------------|
  | 不需要明确的加载模块；功能立即可用                    | 内置模块的变化需要重新编译PHP     |
  |-------------------------------------------------------------------------------------------|
  | 没有使磁盘杂乱的外部文件；什么都驻留在PHP二进制文件中 | PHP二进制文件增多并且消耗更多内存 | 
  |-------------------------------------------------------------------------------------------|
  ```

  当你有一个保持相对不变的可靠的二进制的功能库，要求比平均性能好，或者在你的站点上被许多脚本经常的使用，内置模块是最好的。
必须重新编译PHP的操作很快被速度和易用的好处所补偿。然而，当需要快速的开发小的补充时，内置模块并不是最适合的。
  
Zend引擎：
  当然，扩展同样可以直接在Zend引擎中实现。这是个很好的策略当你需要改变语言行为或直接构建至语言核心的特殊功能。总的来说，不管怎样，
修改Zend引擎应该避免。这里的改变导致与外界不兼容，并且任何人几乎永远不会适应打过补丁的Zend引擎。
修改部分不能与主PHP源文件分开并且下次使用官方源文件库升级会覆盖。因此，这种方法通常被认为是糟糕的实践，由于它的罕见，没有包括在这里。


### 源码排版

  在我们讨论代码问题前，你应该熟悉源代码树能快速浏览PHP文件。这是实现和调试扩展必须具备的能力。
接下来的表格描述了主要目录的内容：

  ```
   ---------------------------------------------------------------------------------------------------
  | 目录                内容                                                                          |
  |---------------------------------------------------------------------------------------------------|
  | php-src           主要的PHP源文件和主要的头文件；这里有所有PHP的API定义，宏，等等（重要）。       |
  |                   其它任何东西都在这个目录下面。                                                  |
  |---------------------------------------------------------------------------------------------------|
  | php-src/ext       动态的和内置模块的仓库；默认情况，这些是与主要源文件树融为一体的官方PHP模块。   |
  |                   从PHP4.0开始，编译这些标准扩展作为动态加载的模块是可能的（至少，是支持的）。    |
  |---------------------------------------------------------------------------------------------------|
  | php-src/main      这个目录包含主要的php宏和定义。（重要）                                         |
  |---------------------------------------------------------------------------------------------------|
  | php-src/pear      PHP扩展和应用仓库(PEAR)。这个目录包含核心PEAR文件。                             |
  |---------------------------------------------------------------------------------------------------|
  | php-src/sapi      包含针对不同服务器抽象层的代码。                                                |
  |---------------------------------------------------------------------------------------------------|
  | TSRM              Zend和PHP的"线程安全资源管理器"(Thread Safe Resource Manager, TSRM)。                                         |
  |---------------------------------------------------------------------------------------------------|
  | ZendEngine2       Zend引擎文件；这里可以找到所有Zend的API定义，宏，等等（重要）。                 |
   ---------------------------------------------------------------------------------------------------
  ```
     
  讨论包含在PHP包中的所有文件超出了本章范围。然而，你应该仔细看一下下面的文件：
    
  * php-src/main/php.h, 在PHP主目录内。这个文件包含大部分PHP的宏和API定义。
  * php-src/Zend/zend.h, 在Zend主目录内。这个文件包含大部分Zend的宏和定义。
  * php-src/Zend/zend_API.h, 同样在Zend目录内，定义了Zend的API。
    
  你应该同样跟随一些这些文件的子包含；例如，和Zend执行相关的，PHP初始化文件支持，和诸如此类。
阅读这些文件之后，花时间浏览一点整个包来看所有文件和模块的依赖关系 - 它们如何相互联系，特别是它们如何利用彼此。
它同样帮你适应PHP编写的编码风格。为了扩展PHP，你应该快速适应这种风格。
    
扩展惯例：
  Zend是利用某些约定建立；为了防止破坏标准，你应该遵循下面部分描述的规则。
    
宏：
  对几乎每一个重要的任务来说，Zend传输预定义的宏非常好用。下面几节中的表格和数字描述了大部分基础函数，结构，和宏。
宏定义能在主要的 zend.h 和 zend_API.h 中找到。我们建议你学完这章后仔细看看这些文件。（尽管你现在可以继续读它们，不是一切都会对你有意义。）
    
内存管理：
  资源管理是至关重要的问题，特别在服务器软件中。最有价值的资源是内存，内存管理应该极度小心地处理。内存管理部分已被抽象到Zend中，
你应该遵守这种抽象的明确原因是：由于这种抽象，Zend获得所有内存分配的完全控制权。
Zend能够确定一个块是否在使用，自动释放未使用的块和丢失引用的块，这样来阻止内存泄漏。使用的函数在下表中：
    
  ```
   -------------------------------------------------------------------------
  | 函数            描述                                                    |
  |-------------------------------------------------------------------------|
  | emalloc()     作为 malloc() 的替代                                      |
  |-------------------------------------------------------------------------|
  | efree()       作为 free() 的替代                                        |
  |-------------------------------------------------------------------------|
  | estrdup()     作为 strdup() 的替代                                      |
  |-------------------------------------------------------------------------|
  | estrndup()    作为 strndup() 的替代。比 estrdup() 更快并且二进制安全。  |
  |               这是推荐使用的函数，如果你在复制字符串之前知道它的长度。  |
  |-------------------------------------------------------------------------|
  | ecalloc()     作为 calloc() 的替代                                      |
  |-------------------------------------------------------------------------|
  | erealloc()    作为 realloc() 的替代                                     |
   -------------------------------------------------------------------------
  ```
  
    译者注：
    malloc()  : 手动分配内存空间，malloc() 不能初始化所分配的内存空间。
    free()    : 手动释放 malloc() 分配的内存空间。
    strdup()  : 字符串拷贝函数，strdup() 内部调用了 malloc() 为变量分配内存，不需要使用返回的字符串时，需要 free() 释放。
    calloc()  : 与 malloc() 相似，但 calloc() 会将所分配的内存空间中的每一位都初始化为零。
    realloc() : 给一个已经分配了地址的指针重新分配空间。
      
  emalloc(), estrdup(), estrndup(), ecalloc() 和 erealloc() 分配内部内存；efree() 释放之前分配的内存。
由 e*() 函数处理的内存被认为是本地当前进程并且由这个进程执行的脚本一结束就被丢弃。
      
  Warning：给生存终止的当前脚本分配常驻内存，你可以使用 malloc() 和 free()。
  这应该只在极其小心的情况下做，然而，只有与Zend API的需求相结合；否则，你将冒内存泄露的风险。
    
  Zend还有线程安全资源管理的特点，来提供对多线程Web服务器更好地原生支持。这需要你为所有的全局变量分配局部结构以便允许并发线程运行。
因为Zend的线程安全模式当它写完后还没有完毕，它还没有广泛地覆盖。
    
目录和文件函数：
  下面的目录和文件函数应该在Zend模块中使用。它们的行为完全像C的副本，但它们在线程级别上提供虚拟工作目录支持。
    
  ```
   ---------------------------------------------------------------------------
  | Zend函数        规则的C函数                                               |
  |---------------------------------------------------------------------------|
  | V_GETCWD()      getcwd()                                                  |
  |---------------------------------------------------------------------------|
  | V_FOPEN()       fopen()                                                   |
  |---------------------------------------------------------------------------|
  | V_OPEN()        open()                                                    |
  |---------------------------------------------------------------------------|
  | V_CHDIR()       chdir()                                                   |
  |---------------------------------------------------------------------------|
  | V_GETWD()       getwd()                                                   |
  |---------------------------------------------------------------------------|
  | V_CHDIR_FILE()  把文件路径作为参数，并把当前工作目录改为那个文件的目录    |
  |---------------------------------------------------------------------------|
  | V_STAT()        stat()                                                    |
  |---------------------------------------------------------------------------|
  | V_LSTAT()       lstat()                                                   |
   ---------------------------------------------------------------------------
  ```
  
字符串处理：
  由Zend引擎处理的字符串和其它如整型，布尔型等有一点不同，字符串不需要额外的内存分配来存储它们的值。
如果想在函数中返回一个字符串，传入一个新的字符串变量到符号表，或做相同的事情，你得确保字符串将占用的内存已提前分配，
使用上述的 e*() 函数来分配。（这也许对你没有多大意义；只是现在把它放在脑子里 - 我们很快就会回来。）
    
复杂的类型：
  复杂的类型像数组和对象需要区别对待。Zend为这些类型提供单独的API - 它们使用Hash表存储。
    
  Note：为了减轻下面源码例子的复杂性，我们首先只用类似整型的简单类型来工作。在本章后面会讨论关于创建更多高级类型。
 
    
### PHP的自动生成系统
  
  PHP4有非常灵活地自动生成系统。所有的模块在ext目录的一个子目录中。除了它自己的源代码外，
每个模块包含一个 config.m4 文件，作为模块配置。
（什么是M4：http://www.gnu.org/software/m4/m4.html 译：http://www.cnblogs.com/farwish/p/4899676.html）
  
  所有这些存根文件都是随着 .cvsignore 自动生成，通过一个 ext 目录内叫 ext_skel 的小shell脚本。
作为参数，它需要你想创建模块的名字。shell脚本接下来随着有关的存根文件创建一个同名的目录。
  
  一步一步，这个过程如下：（ 译者注：加 --no-help 选项，则下面的1到8条提示则不显示出来 ）
  ```
    $ ./ext_skel --extname=my_module
    
     Creating directory my_module
     Creating basic files: config.m4 .cvsignore my_module.c php_my_module.h CREDITS EXPERIMENTAL
     tests/001.phpt my_module.php [done].
    
     To use your new extension, you will have to execute the following steps:
     
     1.  $ cd ..
     2.  $ vi ext/my_module/config.m4
     3.  $ ./buildconf
     4.  $ ./configure --[with|enable]-my_module
     5.  $ make
     6.  $ ./php -f ext/my_module/my_module.php
     7.  $ vi ext/my_module/my_module.c
     8.  $ make
     
     Repeat steps 3-6 until you are satisfied with ext/my_module/config.m4 and
     step 6 confirms that your module is compiled into PHP. Then, start writing
     code and repeat the last two steps as often as necessary.
     （重复3-6步，直到你对ext/my_module/config.m4满意并且第6步确保你的模块已经编译进PHP。接下来，开始编写代码，并在必要时重复最后两步。）
    
  Example #1 默认的config.m4文件：
    ```
    dnl $Id: build.xml 297078 2010-03-29 16:25:51Z degeberg $
    dnl config.m4 for extension my_module

    dnl Comments in this file start with the string 'dnl'.
    dnl Remove where necessary. This file will not work
    dnl without editing.

    dnl If your extension references something external, use with:

    dnl PHP_ARG_WITH(my_module, for my_module support,
    dnl Make sure that the comment is aligned:
    dnl [  --with-my_module             Include my_module support])

    dnl Otherwise use enable:

    dnl PHP_ARG_ENABLE(my_module, whether to enable my_module support,
    dnl Make sure that the comment is aligned:
    dnl [  --enable-my_module           Enable my_module support])

    if test "$PHP_MY_MODULE" != "no"; then
      dnl Write more examples of tests here...

      dnl # --with-my_module -> check with-path
      dnl SEARCH_PATH="/usr/local /usr"     # you might want to change this
      dnl SEARCH_FOR="/include/my_module.h"  # you most likely want to change this
      dnl if test -r $PHP_MY_MODULE/; then # path given as parameter
      dnl   MY_MODULE_DIR=$PHP_MY_MODULE
      dnl else # search default path list
      dnl   AC_MSG_CHECKING([for my_module files in default path])
      dnl   for i in $SEARCH_PATH ; do
      dnl     if test -r $i/$SEARCH_FOR; then
      dnl       MY_MODULE_DIR=$i
      dnl       AC_MSG_RESULT(found in $i)
      dnl     fi
      dnl   done
      dnl fi
      dnl
      dnl if test -z "$MY_MODULE_DIR"; then
      dnl   AC_MSG_RESULT([not found])
      dnl   AC_MSG_ERROR([Please reinstall the my_module distribution])
      dnl fi

      dnl # --with-my_module -> add include path
      dnl PHP_ADD_INCLUDE($MY_MODULE_DIR/include)

      dnl # --with-my_module -> chech for lib and symbol presence
      dnl LIBNAME=my_module # you may want to change this
      dnl LIBSYMBOL=my_module # you most likely want to change this 

      dnl PHP_CHECK_LIBRARY($LIBNAME,$LIBSYMBOL,
      dnl [
      dnl   PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $MY_MODULE_DIR/lib, MY_MODULE_SHARED_LIBADD)
      dnl   AC_DEFINE(HAVE_MY_MODULELIB,1,[ ])
      dnl ],[
      dnl   AC_MSG_ERROR([wrong my_module lib version or lib not found])
      dnl ],[
      dnl   -L$MY_MODULE_DIR/lib -lm -ldl
      dnl ])
      dnl
      dnl PHP_SUBST(MY_MODULE_SHARED_LIBADD)

      PHP_NEW_EXTENSION(my_module, my_module.c, $ext_shared)
    fi
    ```
     
  如果你不了解 M4 文件（现在无疑是一个了解它的好时机），刚开始可能有一点迷惑；但它实际上相当简单。
  
  Note：所有以 dnl 为前缀的将被当做注释且不被解析。
  
  config.m4 文件负责在配置时解析传给 configure 的命令行选项。意思是它需要检查所需要的外部文件并且做相似的配置和步骤。
  
  默认的文件在 configure 脚本中创建两个配置指令：--with-my_module 和 --enable-my_module。
当引用外部文件时使用第一个选项（类似 --with-apache 指令引用Apache目录）。
当使用者需要简单决定是否开启你的扩展时使用第二个选项。不管你使用哪个选项，你应该注释掉另一个，不需要的一个；
那就是，如果使用 --enable-my_module，你应该移除支持 --with-my_module，反之亦然。
  
  默认，ext_skel 创建的 config.m4 文件 接受两个指令并且自动开启你的扩展。开启扩展用 PHP_EXTENSION 宏来做。
当使用者渴望改变包含你的模块到PHP二进制文件中的默认行为（通过明确的说明 --enable-my_module 或 --with-my_module），
把 $PHP_MY_MODULE 改为 “yes”：
  ```
    if test "$PHP_MY_MODULE" == "yes"; then dnl
      Action.. PHP_EXTENSION(my_module, $ext_shared)
    fi
  ```
  
  这需要你每次配置和编译PHP时使用 --enable-my_module。（译者注：把$PHP_MY_MODULE改为yes之后）
  
  Note：每次你改了 config.m4 后一定要运行 buildconf
  
  (译者注：
    PHP5构建系统 - ext_skel脚本：http://php.net/manual/zh/internals2.buildsys.skeleton.php
    组成扩展的文件：http://php.net/manual/zh/internals2.structure.files.php )
  
### 创建扩展
  
  我们起初将以创建一个非常简单的扩展作为开始，它基本上不过实现了一个返回接收到的整型参数的函数。下面的扩展展示了源代码。
  
  Example #2 一个简单的扩展
  ```
    /* include standard header */   # 引入标准头
    #include "php.h"

    /* declaration of functions to be exported */ # 导出函数的声明
    ZEND_FUNCTION(first_module);

    /* compiled function list so Zend knows what's in this module */  # 编译函数列表，这样Zend就知道模块里有什么
    zend_function_entry firstmod_functions[] =
    {
        ZEND_FE(first_module, NULL)
        {NULL, NULL, NULL}
    };

    /* compiled module information */ # 编译的模块信息
    zend_module_entry firstmod_module_entry =
    {
        STANDARD_MODULE_HEADER,
        "First Module",
        firstmod_functions,
        NULL, 
        NULL, 
        NULL, 
        NULL, 
        NULL,
        NO_VERSION_YET,
        STANDARD_MODULE_PROPERTIES
    };

    /* implement standard "stub" routine to introduce ourselves to Zend */  实现引入到Zend的标准存根
    #if COMPILE_DL_FIRST_MODULE
    ZEND_GET_MODULE(firstmod)
    #endif

    /* implement function that is meant to be made available to PHP */  # 实现可用于PHP的函数
    ZEND_FUNCTION(first_module)
    {
        long parameter;

        if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "l", &parameter) == FAILURE) {
            return;
        }

        RETURN_LONG(parameter);
    }
    
  ```
  
  这份代码包含一个完整的PHP模块。我们马上将详细地解释源代码，但首先我们想讨论构建过程。
  （这将允许我们在陷入API讨论前进行不耐烦的实验。）
  
  Note：例子的源码使用PHP4.1.0及以上使用的Zend版本中的一些特性，它不会编译更老的PHP.4.0.x版本。
  
编译模块：
  有两种编译模块的基本方式：
    
    * 使用ext目录提供的“make”机制，同样允许构建动态加载的模块。
    * 手工编译源代码。
    
  第一种方法应该明确优先，后来，截至PHP4.0，这已经被规范化为一个标准的构建过程。
它如此复杂的事实同样是它的缺点，不幸的是 - 它首先难以理解。我们本章后面将提供一个更详细的介绍，但首先让我们用默认文件工作。
第二种方法对那些没有全部可用PHP源文件树，没有访问所有文件权限，或者仅仅想玩弄自己键盘的人来说是不错的。
这些情况应该非常罕见，但为了任务的完整性我们同样会描述这种方法。
  
使用Make编译：
  为了使用标准机制编译样本源代码，复制所有子目录到PHP源代码树的ext目录中。
接下来运行 buildconf，将创建一个更新过的包含新扩展适当选项的 configure 脚本。
默认，所有样本源代码是禁用的，所以你不用害怕破坏你的构建过程。
  
  你运行 buildconf 之后，configure --help 显示下面的额外模块：
  ```
    --enable-array_experiments   BOOK: Enables array experiments
    --enable-call_userland       BOOK: Enables userland module
    --enable-cross_conversion    BOOK: Enables cross-conversion module
    --enable-first_module        BOOK: Enables first module
    --enable-infoprint           BOOK: Enables infoprint module
    --enable-reference_test      BOOK: Enables reference test module
    --enable-resource_test       BOOK: Enables resource test module
    --enable-variable_creation   BOOK: Enables variable-creation module
  ```
  
  更早在Example #1 中展示的模块可以用 --enable-first_module 或 --enable-first_module=yes 来打开。
  
手工编译：
  手工编译模块使用下面的命令：
  
  ```
   -------------------------------------------------------------------------------------------------
  | 动作          命令                                                                              |
  |-------------------------------------------------------------------------------------------------|
  | 编译        cc -fpic -DCOMPILE_DL_FIRST_MODULE-1 -l/usr/local/include -l. -l.. -l../Zend -c -o  |
  |-------------------------------------------------------------------------------------------------|
  | 链接        cc -shared -L/usr/local/lib -rdynamic -o <your_module_file> <your_object_file(s)>   |
   -------------------------------------------------------------------------------------------------
  ```
   
  编译模块的命令指示编译器生成一个位置无关的代码( -fpic 不要漏掉)并额外定义了COMPILE_DL_FIRST_MODULE常量\
来告诉模块代码已经编译成一个动态加载的模块（以上的测试模块；我们将马上讨论）。
经过这些选项，它指定了许多被用作最小设置的标准路径来编译源文件。（译者注：position-independent code, PIC）
  
  Note：在例子中的所有路径是相对于 ext 目录。如果你在其它目录编译，相应地改变路径名。
  需要的项是PHP目录，Zend目录，和（如果必要）模块的放置目录。

  链接命令同样是一个普通的命令，指示连接作为一个动态模块。
  
  你可以在编译命令中加入优化选项，尽管这些在例子中已被忽略（但一些包含在更早部分描述的makefile模板中）。
  
  Note：手工编译和链接成PHP二进制文件中的一个静态模块需要很长的指令，因此不在这儿讨论。（键入所有这些命令效率不是很高。）
  
  
### 使用扩展
  根据你所选择的构建过程，你应该选择结束一个链接到你的Web服务器的的PHP二进制文件（或作为CGI运行），或是一个.so（共享对象）文件。
如果你编译例子文件 first_module.c 作为一个共享对象，你的目标文件应该是 first_module.so。
要使用它，首先你要复制一份到PHP能访问的地方。对于一个简单的测试程序，你可以复制它到 htdocs 目录并试一下Example #3 的源代码。
如果把它编译进了PHP二进制文件，省略掉 dl() 调用，因为模块功能对你的脚本立即可用。
  
  Warning：出于安全原因，你不应该把你的动态模块放在公共的可访问目录内。即使可以使用并且测试简单，
  在生产环境中你应该把它们放到独立的目录内。
  
  Example #3 A test file for first_module.so
  ```
    <?php
      // remove next comment if necessary
      // dl("first_module.so"); 

      $param = 2;
      $return = first_module($param);

      print("We sent '$param' and got '$return'");
    ?>
  ```
  
  调用PHP文件应该输出如下：
  ```
    We sent '2' and got '2'
  ```
  
  如果需要，动态加载的模块使用 dl() 函数加载。函数查找指定的共享对象，加载它，使函数对PHP可用。
模块传输 first_module() 函数，接收一个参数，转换成整型，并返回转换的结果。
  
  如果你已经得到了这个，恭喜！你刚才构建了你的第一个PHP扩展。
