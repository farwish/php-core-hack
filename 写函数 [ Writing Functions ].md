```
/*
|----------------------------------------
| Writing Functions
| @weiChen translate, <farwish@live.com>
| @MIT License
*/
```

  http://php.net/manual/en/internals2.funcs.php
  
  PHP中的函数和方法采用同样的方式，一个方法就是一个有指定作用范围的函数；它们类区域的范围。  
  Hacker 可以在指南的其它章节中读到关于类的入口。本节的目的是给Hacker剖析函数或方法；  
  Hacker将学到如何定义方法，如何接收变量和如何返回变量给phper。  
  
  一个函数的结构不能再简单了：
  
```
PHP_FUNCTION(Hackers_function) {
    /* your accepted arguments here */
    long numer;
    
    /* accepting arguments */
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "1", &number) != SUCCESS) {
      return;
    }
    
    /* do some work on the input */
    number *= 2;
    
    /* set return value */
    RETURN_LONG(number);
}
```

  PHP_FUNCTION(hackers_function) 预处理器指令将引起下面的声明：  
  
  `
  void zif_hackers_function(INTERNAL_FUNCTION_PARAMETERS)
  `
  
  INTERNAL_FUNCTION_PARAMETERS 最为宏被定义，并被解释成下表中的：
  
| 名字和类型              |  描述                                                     | 访问宏              |  
| ------------------------|-----------------------------------------------------------|-------------------- |  
| int ht                  | 用户传递的实际参数数量                                    | ZEND_NUM_ARGS()     |  
| zval* return_value      | 由返回值填充的传递给用户的PHP变量指针，默认类型是 IS_NULL | RETVAL_*, RETURN_*  |  
| zval** return_value_ptr | 当返回的PHP引用设置为一个变量的指针。不建议返回引用。     |                     |  
| zval* this_ptr          | 如果这是一个方法调用，指向PHP变量的是 $this 对象          | getThis()           |  
| int return_value_used   | 标记指示返回值是否被调用者使用                            |                     |  

清楚起见，PHP_FUNCTION(hackers_function) 完整的扩展声明，看起来像这样：

```
void zif_hackers_function(int ht, zval* return_value, zval** return_value_ptr, zval* this_ptr, int return_value_used)
```
