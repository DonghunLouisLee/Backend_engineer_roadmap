# Metadata 
author: Louis Lee   
date: 20201224  
topic: network basic + subnet + cidr
# Introduction

# Summary

모든 컴퓨터나 네트워크 장비, 통신기기들은 인터넷을 사용하기 위해 IP주소를 할당받아 이용하게 됩니다. ip 주소를 공부하다보면 subnet, cidr 등 많은 용어들이 등장하는데 처음부터 차근차근 설명을 해보겠습니다. 

## What is ip address?

먼저 ip주소는 무엇으로 구성되어있는가? 
ip 주소는 network id + host id 입니다. 그러면 network id 랑 host id는 무엇이냐? 
예를 들어 제 방 주소는 서울시 영등포구 영등포1가 러스트아파트 10층 5호입니다.
여기서 host id는 서울시 영등포구 영등포1가 러스트아파트(여기까지 써야 이게 아마 우편주소로 나타내질겁니다) 이고 그 뒤에 10층 5호는 network id 라고 생각하시면 됩니다. 
즉, 우편배달부(라우터)가 우편번호(network id)만 보고도 갈수 있는곳이 있고 우편번호에 도착한 후에 정확한 목적지에 가기 위헤 필요한게 바로 10층 5호(host id) 입니다. 

자 그러면 우리는 평소에 ip 주소를 0.0.0.0 즉 4개의 옥탯으로 구분을 하는것을 알고 있습니다. 그러면 어디서 어디까지가 host id 일까요? 이걸 더 자세히 알기 위해서 먼저 ip 클래스에 대해서 알아봅시다. 

IP Class의 경우 A, B, C, D, E Class로 나누어 Network ID와 Host ID를 구분하게 됩니다. 

A Class의 경우 처음 8bit(1byte)가 Network ID이며, 나머지 24bit(3byte)가 Host ID로 사용됩니다. 비트가 0으로 시작하기에 네트워크 할당은 0~127입니다 . 즉, 128 곳에 가능하며, 최대 호스트 수는 16,777,214개입니다. 

B Class의 경우 처음 16bit(2byte)가 Network ID이며, 나머지 16bit(2byte)가 Host ID로 사용됩니다. 비트가 10으로 시작하기에 네트워크 할당은 16,384 곳에 가능하며, 최대 호스트 수는 65,534개입니다. 

C Class의 경우 처음 24bit(3byte)가 Network ID이며, 나머지 8bit(1byte)가 Host ID로 사용됩니다. 비트가 110으로 시작하기에 네트워크 할당은 2,097,152 곳에 가능하며, 최대 호스트 수는 254개입니다.

그럼 D Class와 E Class에 대해 설명드리도록 하겠습니다. 실제 Network에서 사용되는 Class는 A, B, C Class이며, D Class는 Multicast(멀티캐스트), E Class는 미래에 사용하기 위해 남겨둔 것으로 예약되어 있습니다. 그리고 D와 E Class의 경우 실제 사용되는 경우가 거의 없습니다.

그렇다면 ip 주소만 보고 클래스를 어떻게 구분할까요? 이건 첫번째 옥탯으로만으로도 판별 가능합니다. 각각의 옥탯이 표현가능한 수는 2^0부터 2^7로 0부터 255입니다. 그래서 첫번째 옥탯의 범위를 다음과 같이 구분지어 ip class를 판별하게 됩니다. 

A Class : 0 ~ 127 (0.0.0.0 ~ 127.255.255.255) # 첫 128개
B Class : 128 ~ 191 (128.0.0.0 ~ 191.255.255.255) # 다음 64개
C Class : 192 ~ 223 (192.0.0.0 ~ 223.255.255.255) # 다음 32개 
D Class : 224 ~ 239 (224.0.0.0 ~ 239.255.255.255) # 다음 16개
E Class : 240 ~ 255 (240.0.0.0. ~ 255.255.255.255) # 다음 16개

이걸 이진법으로 표현하면 또 다른 방식으로도 알수 있는데 그건 위 값을 가지고 직접 해보면 쉽게 알수 있기에 생략하겠슴다! 

## CIDR

이렇게 클래스만을 가지고 ip를 할당하다보니 시대가 지날수록 네트워크 통신 기기는 늘어나는데 ip는 부족해지는 현상이 발생했습니다. 그래서 이를 해결하고자 클래스를 보완하는것이 바로 cidr
classless inter domain routing입니다. 이는 바로 밑에 설명이 나오는 서브넷을 사용한 ip할당 체계입니다. 

## subnet?

서브넷이 필요한 이유는? 예를 들어 A 클래스 하나를 제 집에서 사용을 해버리면 어떤 일이 벌어질까요? 네. 엄청난 자원 낭비죠. 할당받은 ip 의 클래스에 따라 사용할수있는 host id 가 달라지는데 이를 다 쓸수 있어야 자원이 낭비되지 않겠죠? 그런 용도로 사용되는게 바로 서브넷입니다. 서브넷을 알기전에 먼저 넷마스크란 개념을 알아야 하는데 별거는 아니고 단순히 뒤에 오는 숫자만큼 왼쪽에서부터 1을 채우고 나머지는 0으로 채운 값입니다. 예를 들어 24라고 하면 1111 1111 1111 1111 1111 1111 0000 0000 이겠죠. 서브넷 계산은 이 넷마스크의 bit를 변경함으로써 이루어집니다. 각 ip 클래스마다 디폴트 서브넷 마스크가 있는데 A,B,C 각각 8,16,24입니다. 이건 물론 디폴트 값이고 변경 가능합니다!

서브네팅을 계산하는 방법은 먼저 ip 주소를 하나 특정하고(ex) 1.1.1.1) subnet mask를 정합니다(ex) 16 = 255.255.0.0 ) 그런 다음에 이 두개를 이진법인 상태에서 and 연산을 하면 되는데 이렇게 되면 network id 가 1.1.0.0 이 되고 나머지가 host id가 됩니다. 여기서 꼭 명심해야 할점은 2진수로 표현하였을 때 Network ID 부분은 1이 연속적으로 있어야 하며, Host ID 부분은 0이 연속적으로 있어야 합니다. 즉, 중간에 1이나 0이 섞이면서 나열될 수 없습니다. 이로 인해 서브넷 마스크는 Network ID를 확장하면서 1bit씩 확보하게 되면 네트워크 할당 가능 수가 2배수로 증가하지만 반대로 호스트 할당가능 수가 2배수로 줄어들게 됩니다. 예를 들어 11111111.11111111.1111111.00000000(255.255.255.0)에서  네 번째 옥탯에 1bit를 확보하면 11111111.11111111.11111111.10000000(255.255.255.128)이 됩니다.

서브넷팅을 통해 Network ID가 확장되므로 인해 할당할 수 있는 네트워크의 수가 늘어납니다. 하지만 네트워크가 분리되므로 인하여 서로가 통신하기 위해서는 라우터를 통하여서만 가능하게 됩니다. 물론 각 네트워크에 속해 있는 호스트들은 같은 영역에 존재하기에 라우터까지 거치지 않고도 통신할 수 있습니다.

# Notes 

http://korean-daeddo.blogspot.com/2015/12/ip.html
http://korean-daeddo.blogspot.com/2016/01/blog-post_26.html
