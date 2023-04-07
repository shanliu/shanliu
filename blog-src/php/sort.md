# sort

```php
<?php
//版本1
function q2sort(&$arr){
    if(count($arr)>1){
        $k=$arr[0];
        $x=array();
        $y=array();
        $_size=count($arr);
        for($i=1;$i<$_size;$i++){
            if($arr[$i]<=$k){
                $x[]=$arr[$i];
            }elseif($arr[$i]>$k){
                $y[]=$arr[$i];
            }
        }
        $x=q2sort($x);
        $y=q2sort($y);
        return array_merge($x,array($k),$y);
    }else{
        return$arr;
    }
}
$a=array(13,22,12,17,32,23,12,25,22,32,12);
$a=q2sort($a);
print_r($a);
//----------------------------------------------------------------------------
//版本二
function start_move($c,&$a,$start,$end){
    while(true){
        if($start==$end){$a[$start]=$c;break;}
        if($a[$start]>$c){
            $a[$end]=$a[$start];
            return end_move($c,$a,$end-1,$start);
        }else{ $start++;}
    }
    return $start;
}
function end_move($c,&$a,$end,$start){
    while(true){
        if($start==$end){$a[$start]=$c;break;}
        if($a[$end]<$c){
            $a[$start]=$a[$end];
            return start_move($c,$a,$start+1,$end);
        }else{ $end--;}
    }
    return $start;
}
function q1sort(&$a,$start,$end){
    if($start>=$end)return ;
    $c=end_move($a[$start],$a,$end,$start);
    q1sort($a,0,$c-1);
    q1sort($a,$c+1,$end);
}
$a=array(13,22,12,17,32,23,12,25,22,32,12);
q1sort($a,0,count($a)-1);
print_r($a);
```
