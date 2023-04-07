# re2c\&bison

BNF:

非终结符: 终结符|非终结符

流程

yyparse(bison 入口) ->(内部) yylex ->(内部) scan(re2c 正则分词,返回token) -> 语法分析 ->构建ast

添加PHP语法:
```
| variable '=' '&' variable { \$$ = zend\_ast\_create(ZEND\_AST\_ASSIGN\_REF, $1, $4); }
```
//下面添加 
```
| variable '=' '?' expr { \$$ = zend\_ast\_create(ZEND\_AST\_ASSIGN, $1, zend\_ast\_create(ZEND\_AST\_COALESCE, $1, $4)); }
```
支持 $a=?1111; 当$a 未定义时赋值
