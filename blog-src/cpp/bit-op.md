# 加减乘除位运算


> 加法 = 异或＋(并<<1)

```c
// 递归写法
int add(int num1, int num2){
    if(num2 == 0)
        return num1;
    int sum = num1 ^ num2;
    int carry = (num1 & num2) << 1;//并后为1表示会进位
    return add(sum, carry);//carry>0表示有进位操作，加入计算
}
​
// 迭代写法
int add(int num1, int num2){
    int sum = num1 ^ num2;
    int carry = (num1 & num2) << 1;  
    while(carry != 0){
        int a = sum;
        int b = carry;
        sum = a ^ b;
        carry = (a & b) << 1;  
    }
    return sum;
}
```

> 减法 ＝ 加负数

```c
int substract(int num1, int num2){
    int subtractor = add(~num2, 1);// 先求减数的补码（取反加一）
    int result = add(num1, subtractor); // add()即上述加法运算　　
    return result ;
}
```

> 乘法 ＝ 多个数累加

```c
//v1
int multiply(int a, int b){ 
    // 取绝对值　　    
    int multiplicand = a < 0 ? add(~a, 1) : a;    
    int multiplier = b < 0 ? add(~b , 1) : b;// 如果为负则取反加一得其补码，即正数　　    
    // 计算绝对值的乘积　　    
    int product = 0;    
    int count = 0;    
    while(count < multiplier) {        
        product = add(product, multiplicand);        
        count = add(count, 1);// 这里可别用count++，都说了这里是位运算实现加法　　    
    }    
    // 确定乘积的符号　　    
    if((a ^ b) < 0) {// 只考虑最高位，如果a,b异号，则异或后最高位为1；如果同号，则异或后最高位为0；　　　　        
        product = add(~product, 1);    
    }    
    return product;
}
//v2
int multiply(int a, int b) {　　
    //将乘数和被乘数都取绝对值　
    int multiplicand = a < 0 ? add(~a, 1) : a; 　　
    int multiplier = b < 0 ? add(~b , 1) : b;　　
    　
    //计算绝对值的乘积　　
    int product = 0;　　
    while(multiplier > 0) {　　　　
        if((multiplier & 0x1) > 0) {// 每次考察乘数的最后一位　　　　
            product = add(product, multiplicand);　　　　
        } 　　　　
        multiplicand = multiplicand << 1;// 每运算一次，被乘数要左移一位　　　　
        multiplier = multiplier >> 1;// 每运算一次，乘数要右移一位（可对照上图理解）　　
    } 　　
    //计算乘积的符号　　
    if((a ^ b) < 0) {　　　　
        product = add(~product, 1);　　
    } 　　
    return product;
}
```