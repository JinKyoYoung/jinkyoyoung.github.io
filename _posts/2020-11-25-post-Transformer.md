---
title: "Transformer"
categories:
  - AI
tags:
  - ai
---

## Transformer.
- 참고 사이트 : https://wikidocs.net/31379

### .
- dmodel : 압축백터의 크기(워드임배딩 : Win 의 백터 열크기)
- num_layers : 인코더 6층, 디코더 6층으로 쌓는다.
- num_heads : 멀티헤더 구조, RNN보다 좋다.
- dff : 완전연결 신경망 (피드 포워드 신경망)

RNN은 순차적으로 받는 특성이 있어, 포지셔널 인코딩을 더해서 사용하는 방식으로 한다.

PE() = sin() : 짝수만 태우고,
PE() = cos() : 홀수만 태운다.

i 는 열을 의미한다.

[:, tf.newaxis] 는 reshape 이랑 동일하다.(열기준으로)
[tf.newaxis, :] 는 행기준으로 reshape을 한거다.

position : 50개, i : 128개
128개가 get_angles() 함수내에서 통으로 계산이 되는거다.

### Multi-head self-Attention
1. 인코더

Q, K, V 벡터 얻기:
dmodel=512 를 1/8 로 줄여서 64의 차원을 가지도록 한다.

Dense 는 세로축을 말하고 여기서는 afine층이다.
units 수는 뉴런의 숫자를 말한다. (뉴런 자체는 행렬이다.)

### self attention 을 구하는 방법.
멀티헤더 벡터는 쿼리를 동일한 크기로 나눠서 들어가도록 하는 것이다!
(4차원 텐서의 계산이 어떻게 되는가를 알아야 한다.)

query 는 이미 4차원 형태다. (1, 4, 40, 32) 40 x 32를 해서 워드벡터를 구하고
4차원 텐서의 matmul 하고 전치 한다.
(a, b, transpose_b=True) 를 matmul 은 query 와 key 만 하고 ..
이걸 모르겠으니.. 부븐 코드를 돌려봐야 한다.
코드 169
(1,3,8)을 4개로 쿼리 쪼개면, (1,4,3,2) 가 되는 거다.
q와 k 가 tf.matmul이 되는 지 확인 해봐야 한다.
(1,4,3,2)(1,4,3,2) transpose_b=True 를 하면
*3차원 안에 있는 2차원들만 전치가 돼서 계산이 되는거다!!!
(1,4) (3,2)(3,2).T 이기 때문에 전치를 해서 (1,4) (3,2)(2,3) 이되고
결론 적으로 (1,4,3,3) 이 되는 것이다..

query         key.T           matmul_qk
(1,4,40, 40)(1,4,32,40)  = (1,4,40,40)
matmul        value           scaled_attention
(1,4,40, 40)(1,4,40, 32)  = (1,4,40,32)
(batch_size, query 의 문장 길이, num_heads, d_model/num_heads)
(1,40,4,32)
reshape 을 통해 (1,40,128) 이 되고 새로운 W0은 (d_model, dv x num_heads) 를 matmul 한게
Multiful_head attention matrix 가 된다!!!

(1,40,128)(1,128,128) 최종적인 형상은 (40, 128)
