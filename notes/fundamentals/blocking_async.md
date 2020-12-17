# Metadata 
author: Louis Lee   
date: 20201217  
topic: blocking/non_blocking and sync/async
# Introduction

blocking, non_blocking
# Summary

개발을 배우다보면 동기, 비동기, blocking, non-blocking 이 네가지 단어들을 여러가지 상황에서 듣게 된다. 원론적으로는 이해한것 같으면서도
다른 언어를 배울때마다 각 언어가 이를 구현하는 방식이나 혹은 개별 라이브러리들이 이를 구현하는 방식이 달라 매번 헷갈리기 정말 쉬운 주제이다. 그래서
이 주제를 한번에 깨끗하게 정리하는게 이 글의 목표다. 동기/비동기를 보기 전에 먼저 blocking, non-blocking에 대해 알아보자. 

## blocking/non-blocking

호출된 함수가 바로 리턴을 해서 호출한 함수에게 제어권을 넘겨주고, 호출한 함수가 다른 일을 할 수 있도록 기회를 주면 non-blocking 이다!
반대로, 호출된 함수가 자신이 작업을 모두 마칠때까지 제어권을 넘겨주지 않는다면 기다리게 만들면 blocking이다!

i/o 작업을 예로 들어보자. 가상의 함수 blocking_io(), non_blocking_io() 두개가 있을때 가상의 러스트 코드를 쓴다면...
```
fn blocking_io(){
    // some io work
    println!("1");
}

fn non_blocking_io(){
    // some io work
    println!("1");
}

fn io1(){
    println!("0"); 
    blocking_io();
    println!("2");
}

fn io2(){
    println!("0"); 
    non_blocking_io();
    println!("2");
}
```
요런 식인데 여기서 우리는 io1은 0,1,2 순서대로 출력하고 io2 는 0,2,1로 출력되는걸 볼수 있다! 

## sync/async

호출되는 함수에게 callback 을 전달해서, 호출되는 함수의 작업이 완료되면 호출되는 함수가 전달받은 callback 을 실행하고 호출하는 함수는 작업 완료 여부를 신경쓰지 않으면 async이다. 반대로 호출하는 함수가 호출되는 함수의 작업 완료 후 리턴을 기다리거나 또는 호출되는 함수로부터 리턴 받더라도 작업 완료 여부를 계속 신경쓰면 sync 이다. 
async를 구현하는 방식은 대표적으로 callback, await 구문이 있는데 callback은 구조가 매우 단순해서 한눈에 알아보기가 쉽다! 
예를 들어본다면 

```
functon non_blocking(){
    console.log("0"); 
    setTimeout(function()  { console.log("1"); }, 100); 
    console.log("2")
}

```
러스트는 await 를 쓰는걸로 굳어져서 부득이하게 js 의 예시를 들고 왔는데, 이 함수의 출력은 0, 2, 1이다! 

await syntax 는 이에 비해 조금 더 복잡하다. 
예시를 한번 들어보자. 

```
async fn non_blocking_io(){
    // some io work
    println!("1");
}
async fn io(){
    println!("0"); 
    non_blocking_io().await;
    println!("2");
}
```

여기서의 출력은? 0,1,2다! 음? 결과가 위에 blocking_io 함수와 같지 않은가? 그러면 이건 blocking code 인가? 아니다. async await 함수를 이해하기 위해서는 정확히 blocking 이 무엇을 blocking 하는지 알아야 한다. blocking을 설명할때 호출하는 함수라고 위에서 정의했는데 정확히 표현을 하자면 호출하는 함수를 실행하고 있는 thread이다. 즉 sync 코드에서는 thread 자체가 더 이상의 작업을 진행하지 못하는 상황에 있는것이다. 그럼 async/await는 무엇인가? async 를 쓸때는 대부분 언어가 스레드를 효율적으로 관리하기 위해 event loop 기반의 각각의 task manager 들을 가지고 있다. rust에는 tokio, golang 은 goroutine 이 대표적인 예시이다. 이 놈들을 기존 스레드 기반 작업을 더 작은 단위인 task 단위로 쪼개서 일을 처리한다. 즉 많은 수의 task들이 한정된 스레드에서 순서대로 실행되는 형태인데 await syntax 들은 이 작은 task들을 스레드에서 잠시 빼준다. 즉 await 함으로써 스레드는 다른 task들을 계속 실행할수 있게 되고 non-blocking이 이루어지게 된다! 


# Notes 