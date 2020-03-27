---
title: "Solução OBI 2018 Cinco"
date: 2020-02-05
tags: ["OBI"]
categories: ["Português"]
---
[Enunciado](https://olimpiada.ic.unicamp.br/pratique/pu/2018/f3/cinco/)  
Podemos criar um algoritmo guloso simples, trocar sempre (em ordem):

1. O dígito mais significativo que for trocado por um número maior do que ele  
2. (na falha do primeiro) O dígito menos significativo que for trocado por um número menor do que ele  

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {

    int n;
    scanf("%d",&n);
    int arr[n];

    for(int i=0;i<n;i++) scanf("%d",&arr[i]);

    for(int i=0;i<n;i++) {
        if(arr[i]<arr[n-1]&&(arr[i]==0||arr[i]==5)) {
            swap(arr[i],arr[n-1]);
            for(int j=0;j<n;j++) printf("%d%c",arr[j],j==n-1?'\n':' ');
            exit(0);
        }
    }

    for(int i=n-1;i>=0;i--) {
        if(arr[i]==0||arr[i]==5) {
            swap(arr[i],arr[n-1]);
            for(int j=0;j<n;j++) printf("%d%c",arr[j],j==n-1?'\n':' ');
            exit(0);
        }
    }

    puts("-1");

    return 0;
}

```
