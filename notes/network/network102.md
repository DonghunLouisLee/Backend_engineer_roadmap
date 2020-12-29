# Metadata 
author: Louis Lee   
date: 20201228  
topic: public ip vs private ip, NAT

# Introduction

저번 network101 에 이어 이번에는 public ip 와 private ip 에 대해 알아보겠습니다. 

# Summary

## public ip and private ip

ip 에 대해서 공부하다보면(혹은 vpn setting을 건드리다 보면) public ip 와 private ip 에 대해 듣게 되는데 이에 대해 조금 더 자세히 알아보는 것도 나쁘지 않습니다. 
먼저 public ip는 우리가 인터넷에서 사용되는 글로벌 ip 입니다. 마치 각 컴퓨터(혹은 라우터)에 대한 주소라고 생각하면 됩니다. public ip 은 인터넷 상에서 라우팅이 되면 이걸 통해 전세계 인터넷이 서로 연결되었다고 생각하면 됩니다. 자 그렇다면 만약에 세상에 존재하는 모든 컴퓨터(인터넷이 가능한 머신)이 public ip를 할당받으면 어떻게 될까요? 네 저번 편에서 살펴았던것 처럼 무수히 많은 수의 컴퓨터 숫자로인해 public ip가 부족해지는 상황이 발생합니다. 그래서 가능하다면 public ip 하나를 여러개의 컴퓨터가 나누어 쓰면 이 문제가 해결되는데 이를 해결하기 위해서 라우터(그리고 private ip)가 존재합니다. private ip는 말 그대로 외부에 노출되지 않는 같은 네트워크를 공유하는 컴퓨터들에게 라우터가 나누어주는 ip 입니다. 즉, 라우터를 통하면 public ip가 private ip로 바뀌고 이 private ip는 외부에서 직접적으로는 접속이 불가능하게 됩니다. public ip가 private ip로 바뀌는것은 뒤에 NAT에서 설명을 추가로 하고 이 부분에서는 그러면 private ip 할당에 대해 조금 덜 설명을 해보겠습니다. 

ip 할당에 대해 조금 더 자세히 알고 싶으신 분은 network101 전글을 보고 오시면 더 이해가 쉽게 되는데 일단 private ip 를 위해 할당된 구간은 4개가 있습니다. 

1. 10.0.0.0 to 10.255.255.255 —> 10.0.0.0 network with a 255.0.0.0 or an /8 (8-bit) mask
2. 172.16.0.0 to 172.31.255.255 —> 172.16.0.0 network with a 255.240.0.0 (or a 12-bit) mask
3. 192.168.0.0 to 192.168.255.255 range, which is a 192.168.0.0 network masked by 255.255.0.0 or /16
4. 100.64.0.0 to 100.127.255.255 with a 255.192.0.0 or /10 network mask

이 4개 구간이 private ip를 위해 할당된 ip 인데 여기서 4번은 CGN(Carrier-Grade NAT)은 큰 통신사들이 사용하는 ip여서 대부분 회사 혹은 집에서 로컬로 private ip 를 할당하면 1,2,3 중 하나를 보시게 됩니다. 위에 언급했던것처럼 인터넷에서 직접 저 private ip로 접속은 불가능합니다. 다만 한가지 유의해야 할점은 한 네트워크에서 private ip는 unique 해야 한다는 점입니다. 

## NAT 

자 그러면 public ip <-> private ip가 어떻게 치환이 되는지 궁금해하시는 분들이 있는데 이를 알기 위해서는 NAT(Network Address Translation)에 대해 조금 더 깊게 알 필요가 있습니다. 예를 들기 위해 저희 집 라우터는 public ip 11.11.11.11을 가지고 있고 제 맥북은 10.0.0.1, 윈도우컴은 10.0.0.2의 private ip를 가지고 있다고 가정을 합니다. 그리고 제가 접속하려는 google 의 ip는 12.12.12.12라고 가정을 하겠습니다. 그러면 제 맥북에서는 구글에 get request를 요청하면 먼저 제 라우터에서 private ip가 pulbic ip로 바뀌고 이 바뀐 public ip로 구글에 요청을 하게 됩니다. 즉, 구글 입장에서는 11.11.11.11에서부터 접속 요청이 온것이죠. 그러면 라우터는 어떻게 private ip를 public ip 로 바꾼것일까요? 이 변환과정에서 NAT가 사용되는데 대표적인 방법들을 소개하겠습니다. 

1. SNAT(Static Network Translation)

Static NAT – In this, a single unregistered (Private) IP address is mapped with a legally registered (Public) IP address i.e one-to-one mapping between local and global address. This is generally used for Web hosting. These are not used in organisations as there are many devices who will need Internet access and to provide Internet access, the public IP address is needed.

2. DNAT(Dynamic Network Translation)

Dynamic NAT – In this type of NAT, an unregistered IP address is translated into a registered (Public) IP address from a pool of public IP address. If the IP address of pool is not free, then the packet will be dropped as an only a fixed number of private IP address can be translated to public addresses.

Suppose, if there is a pool of 2 public IP addresses then only 2 private IP addresses can be translated at a given time. If 3rd private IP address wants to access Internet then the packet will be dropped therefore many private IP addresses are mapped to a pool of public IP addresses. NAT is used when the number of users who wants to access the Internet is fixed. This is also very costly as the organisation have to buy many global IP addresses to make a pool.

3. Port Forwarding

This is also known as NAT overload. In this, many local (private) IP addresses can be translated to a single registered IP address. Port numbers are used to distinguish the traffic i.e., which traffic belongs to which IP address. This is most frequently used as it is cost-effective as thousands of users can be connected to the Internet by using only one real global (public) IP address.

# Notes 
https://help.keenetic.com/hc/en-us/articles/213965789-What-is-the-difference-between-a-public-and-private-IP-address-#:~:text=Range%20from%2010.0.,255.255%20%E2%80%94%20a%2010.0.
https://www.geeksforgeeks.org/types-of-network-address-translation-nat/
https://www.geeksforgeeks.org/network-address-translation-nat/