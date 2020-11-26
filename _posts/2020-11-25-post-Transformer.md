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
