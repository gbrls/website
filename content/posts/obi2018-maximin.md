---
title: "Solução OBI 2018 Maximin"
date: 2020-02-05
tags: ["OBI"]
categories: ["Português"]
---
[Enunciado](https://olimpiada.ic.unicamp.br/pratique/pu/2018/f3/maximin/)  
Como o tamanho máximo do vetor é \\(10^5\\) podemos ordená-lo. Fazendo isso podemos ver que o número que estamos procurando está
entre dois números vizinhos no vetor ou está em alguma das extremidades.

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n,r,l;
    scanf("%d%d%d",&n,&r,&l);
    int arr[n];
    for(int i=0;i<n;i++) scanf("%d",&arr[i]);

    sort(arr,arr+n);

    int dif=0;
    for(int i=1;i<n;i++) {
        int mid=(arr[i]+arr[i-1])/2;
        if(mid>=r&&mid<=l) dif=max(dif,min(abs(mid-arr[i]),abs(mid-arr[i-1])));
    }

    if(l>arr[n-1]) dif=max(dif, l-arr[n-1]);
    if(r<arr[0]) dif=max(dif,arr[0]-r);

    printf("%d\n",dif);

    return 0;
}
```
