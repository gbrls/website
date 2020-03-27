---
title: "Solução OBI 2018 Baldes"
date: 2020-02-05
tags: ["OBI"]
categories: ["Português"]
---
[Enunciado](https://olimpiada.ic.unicamp.br/pratique/pu/2018/f3/baldes/)  
Na minha primeira leitura, vi que esta é uma questão clássica de segtree. Um vetor de tamanho \\(10^5\\) e \\(10^5\\) queries dentro desse intervalo. Iremos fazer uma segtree para o mínimo num intervalo e outra para o máximo, sendo a resposta da query em um intervalo o max-min. Para simplificar isso, podemos guardar as duas árvores apenas em um vetor de pairs.


Porém após implementar a primeira solução e não funcionar fiz uma segunda leitura e percebi que o max e o min não podem estar no mesmo balde (na mesma posição no vetor). Então troquei o pair por uma struct, para guardar de qual balde o max e o min vieram. Quando o min e o max estiverem no mesmo balde temos uma certeza, um dos dois faz parte da solução. Nesse caso podemos testar, o query sem o min, mas com o max e o query sem o max, mas com o min.

```cpp
#include <bits/stdc++.h>
typedef struct ii {
    int first;
    int second;
    int i;
    int j;
} ii;
using namespace std;

const int MAX = 5e5+50;
const int inf = 0x3f3f3f3f;

ii tree[MAX]={};
int arr[MAX];

ii merge(ii a, ii b) {

    ii c = {min(a.first,b.first),max(a.second,b.second),-1,-2};

    if(c.first==a.first) c.i=a.i;
    else c.i=b.i;

    if(c.second==a.second) c.j=a.j;
    else c.j=b.j;

    return c;
}

void build(int pos, int i, int j) {
    if(i==j) {
        tree[pos]={arr[i],arr[i],i,i};
        if(arr[i]==0) {
            tree[pos]={inf,-inf,-1,-2};
        }
    } else {
        build(2*pos,i,(i+j)/2);
        build(2*pos+1,(i+j)/2+1,j);

        tree[pos]=merge(tree[2*pos],tree[2*pos+1]);
    }
}

void update(int pos, int i, int j, int target, ii val) {
    if(i==j) {
        if(tree[pos].first!=0&&tree[pos].second!=0) {
            tree[pos]=merge(tree[pos],val);
        } else {
            tree[pos]=val; 
        }
    } else {
        if(target <= (i+j)/2) update(2*pos,i,(i+j)/2,target,val);
        else update(2*pos+1,(i+j)/2+1,j,target,val);

        tree[pos]=merge(tree[2*pos],tree[2*pos+1]);
    }
}

void force_update(int pos, int i, int j, int target, ii val) {
    if(i==j) {
        tree[pos]=val; 
    } else {
        if(target <= (i+j)/2) force_update(2*pos,i,(i+j)/2,target,val);
        else force_update(2*pos+1,(i+j)/2+1,j,target,val);
        tree[pos]=merge(tree[2*pos],tree[2*pos+1]);
    }
}

ii query(int pos, int i, int j, int a, int b) {
    if(i>b||j<a) {
        return {inf,-inf, -1, -2};
    } else if(i>=a&&j<=b) {
        return tree[pos];
    } else {
        return merge(query(2*pos,i,(i+j)/2,a,b),query(2*pos+1,(i+j)/2+1,j,a,b));
    }
}

int main() {

    int n,m;
    scanf("%d%d",&n,&m);

    for(int i=0;i<n;i++) scanf("%d",&arr[i]);

    build(1,0,n);

    for(int i=0;i<m;i++) {
        int op;
        scanf("%d",&op);

        if(op==1) {
            int w,p;
            scanf("%d%d",&w,&p);
            --p;
            update(1,0,n,p,{w,w,p,p});
        } else {
            int a,b;
            scanf("%d%d",&a,&b);
            --a,--b;
            ii p = query(1,0,n,a,b);

            if(p.i==p.j) {
                ii tmp = p;
                int ans=0;
                force_update(1,0,n,p.i,{p.first,-inf,p.i,p.j});
                ii A = query(1,0,n,a,b);
                force_update(1,0,n,p.i,{inf,p.second,p.i,p.j});
                ii B = query(1,0,n,a,b);

                ans=max(A.second-A.first,B.second-B.first);
                printf("%d\n",ans);

                update(1,0,n,p.i,p);
            } else {
                printf("%d\n",p.second-p.first);
            }
        }
        
    }

    return 0;
}
```
