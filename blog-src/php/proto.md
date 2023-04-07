# protobuf

//composer.json
```
{
    "autoload":{
        "psr-4": {
            "" : "src/proto/"
        }
    }
}
```

```php
$user=new Foo\User();
//联合体 只可能存在一个
$user->setOne1(1);
$user->setOne2(2);
$user->setUserId(100);
$user->setUserHeight(1.1);
$user->setUserMoney(11);
$user->setUserName("namne");
$user->setUserType(Foo\UserType::HIG);
$ui=new Foo\UserItem();
$ui->setItemId(1);
$ui->setItemName("item");
$ui1=new Foo\UserItem();
$ui1->setItemId(1);
$ui1->setItemName("item");
$uis=array($ui,$ui1);
$user->setUserItems($uis);
$arr=array(1=>1111);
$user->setAttr($arr);
//生成出二进制字符
$k=($user->serializeToString());
//生成出JSON字符
// echo $user->serializeToJsonString();
// exit;
//还原为对象
$buser=new Foo\User();
$buser->mergeFromString($k);
var_dump($buser->getOne1());
var_dump($buser->getOne2());
var_dump(iterator_to_array($buser->getAttr()));
var_dump($buser->getUserHeight());
var_dump($buser->getUserId());
var_dump(iterator_to_array($buser->getUserItems()));
var_dump($buser->getUserMoney());
var_dump($buser->getUserName());
var_dump($buser->getUserType());
```