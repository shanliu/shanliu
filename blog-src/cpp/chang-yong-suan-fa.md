# 常用算法

1. HASHTABLE

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

typedef struct _hashnode{
    int val;
    char * key;
    struct _hashnode * next;
} hashnode;

//存储hash表大小
#define HASHTABLE_MAX 1000
//form php source
static inline unsigned int hashtable_hash_str(const char *arKey)
{
    register unsigned int hash = 5381;
    unsigned int nKeyLength=strlen(arKey);
    /* variant with the hash unrolled eight times */
    for (; nKeyLength >= 8; nKeyLength -= 8) {
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
        hash = ((hash << 5) + hash) + *arKey++;
    }
    switch (nKeyLength) {
        case 7: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 6: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 5: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 4: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 3: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 2: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
        case 1: hash = ((hash << 5) + hash) + *arKey++; break;
        case 0: break;
    }
    return hash;
}


hashnode * hashtable_create(){
    hashnode * hashs=(hashnode *)malloc(HASHTABLE_MAX * sizeof(hashnode *));
    memset(hashs, 0, HASHTABLE_MAX * sizeof(hashnode *));
    return hashs;
}

void hashtable_set(hashnode* hashlist[],const char * b,int val){
    unsigned int pos=hashtable_hash_str(b)%HASHTABLE_MAX;
    hashnode * hashp= hashlist[pos];
    while(hashp){
        if(strcmp(hashp->key,b)==0){
            break;
        }
        hashp=hashp->next;
    }
    if(!hashp){
        hashnode * p=(hashnode *)malloc(sizeof(hashnode));
        memset(p,0,sizeof(hashnode));
        char * ikey=(char *)malloc(sizeof(char)*(strlen(b)+1));
        strcpy(ikey,b);
        p->key=ikey;
        p->val=val;
        p->next=hashlist[pos];
        hashlist[pos]=p;
    }else{
        hashp->val=val;
    }
}

int hashtable_get(hashnode* hashlist[],const char * b,int *r){
    unsigned int pos=hashtable_hash_str(b)%HASHTABLE_MAX;
    hashnode * hashp= hashlist[pos];
    while(hashp){
        if(strcmp(hashp->key,b)==0){
            *r=hashp->val;
            return 1;
        }
        hashp=hashp->next;
    }
    return 0;
}

void hashtable_unset(hashnode* hashlist[],const char * b){
    unsigned int pos=hashtable_hash_str(b)%HASHTABLE_MAX;
    hashnode * hashp= hashlist[pos];
    hashnode * phashp=NULL;
    while(hashp){
        if(strcmp(hashp->key,b)==0){
            if(phashp)
                phashp->next=hashp->next;
            else
                hashlist[pos]=hashp->next;
            free(hashp->key);
            free(hashp);

        }
        phashp=hashp;
        hashp=hashp->next;
    }
}
void hashtable_free(hashnode* hashlist[]){
    int i=0;
    while(i<HASHTABLE_MAX){
        hashnode * temp=hashlist[i];
        while(temp){
            hashnode * tt=temp->next;
            free(temp->key);
            free(temp);
            temp=tt;
        }
        i++;
    }
    hashlist=NULL;
}

int main(){
    //使用
    hashnode * ht=hashtable_create();
    hashtable_set(ht,"abc",154544);
    int r=0;
    if(hashtable_get(ht,"abc",&r))
        printf("%d",r);
    hashtable_unset(ht,"abc");
    if(hashtable_get(ht,"abc",&r))
        printf("%d",r);
    //释放所占内存
    hashtable_free(ht);
    return EXIT_SUCCESS;
}
```
