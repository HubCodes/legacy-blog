---
layout: post
tags: test
comments: true
---

TCP 스택은 애플리케이션 데이터 스트림을 네트워크 상으로 전송할 때 세그먼트라고 부르는 적절한 단위로 쪼개야 한다. 이때 이 쪼개는 작업을 커널에서 수행하면 약간이나마 커널 cpu 시간을 잡아먹게 되고, TCP 세그먼트를 한 두개 전송하고 받는 것이 아니므로 이는 상당한 오버헤드를 유발하게 된다. 그래서, cpu는 다른 일 하기도 바쁘니까 그런 일은 NIC한테 맡기자고 생각하게 된 결과물이 tcp segmentation offloading이다.

[이 링크에 따르면](https://lwn.net/Articles/9129/) 이걸 적용한 다음 확실히 cpu 점유율이 줄어들었고, 약간 성능도 증가한 모양새다.
이와 비슷하게, 가상 헤더를 만들고 세그먼트에 대한 체크섬을 계산하는 작업도 NIC가 수행하게끔 한다. (checksum offloading)
내 컴퓨터에서는 다음 명령들로 확인할 수 있었다.

```sh
[~]$ ethtool -k enp4s0 | grep tcp-segmentation-offload
tcp-segmentation-offload: on
[~]$ ethtool -k enp4s0 | grep -e "[rt]x-checksumming" 
rx-checksumming: on
tx-checksumming: on
```

{% include disqus_comments.html %}
