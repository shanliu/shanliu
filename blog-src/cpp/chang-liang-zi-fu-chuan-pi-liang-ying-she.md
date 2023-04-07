# PHP中常量字符串批量映射

```c
#define ZEND_KNOWN_STRINGS(_) \
    _(ZEND_STR_FILE,                   "file")
// 上面为字符串映射列表定义
typedef enum _zend_known_string_id {
#define _ZEND_STR_ID(id, str) id,
    ZEND_KNOWN_STRINGS(_ZEND_STR_ID)
#undef _ZEND_STR_ID
    ZEND_STR_LAST_KNOWN
} zend_known_string_id;
static const char *known_strings[] = {
#define _ZEND_STR_DSC(id, str) str,
        ZEND_KNOWN_STRINGS(_ZEND_STR_DSC)
#undef _ZEND_STR_DSC
        NULL
};
//遍历
for (int j = 0; j < ZEND_STR_LAST_KNOWN; j++) {
    printf("%s",known_strings[j]);
}
//指定
printf("%s",known_strings[ZEND_STR_FILE]);
```
