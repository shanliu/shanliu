# php7-feature

1.  命名空间简化

    ```
    use a\b\{a,b};
    ```
2.  ?? 语法

    ```
    $username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
    //不存在用下一个值
    ```
3.  函数强类型

    ```
    function foobar(float $abc): bool {
     return $abc+1;
    }
    echo foobar(0.5);
    ```
4.  new类直接使用

    ```
    class s {public $x = 1;}
    (new s())->x;
    ```
5.  闭包调用类调用

    ```
    class S{private $x=10;};
    $getX = function() {return $this->x;};
    echo $getX->call(new s);
    ```
6.  同类型不定参数

    ```
    function sumOfInts(int ...$ints)
    {
     return array_sum($ints);
    }
    var_dump(sumOfInts(2, '3', 4.1));
    ```
7.  生成器yield

    ```
    function xrange() {
     echo "aa";
     $a=yield 1;
     $b=yield 2;
    }
    $x=xrange();
    //调用时候才运行xrange内容 $x 为Generator
    //调用1
    foreach ($x as $num) {
     echo $num, "\n";
    }
    //调用2
    $x->send("a");
    $x->send("b");
    ```
8.  匿名类

    ```
    echo (new class{public $a=10;})->a;
    ```
9.  数组常量支持

    ```
    define("A", ["C"=>"a","bb"]);
    echo A['C'];
    ```
10. Trait 代码片段

    ```
    trait trait1 {
    public function a() { echo "1a";}
    public function b() { echo "1b";}
    }
    trait trait2 {
    public function a() { echo "2a"; }
    }
    trait trait3 {
    use trait1, trait2 {
        trait2::a insteadof trait1;
    }
    }
    class cls1{
    use trait1, trait2 {
        //用 trait2::a 代替 trait1的a
        trait2::a insteadof trait1;
        //方法重命名
        trait1::b as c;
    }
    }
    class cls2 {
    use trait1;
    }
    ```
