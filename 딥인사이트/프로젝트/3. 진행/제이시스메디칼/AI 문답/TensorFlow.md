---
title:
source:
author:
published:
created: 2025-09-07
description:
tags:
---
TensorFlow Lite(현재 명칭 LiteRT)는 ==모바일, 임베디드, IoT 기기 등 에지 기기에서 인공지능 모델을 빠르게 실행하기 위해 구글이 만든 경량 프레임워크==입니다. 서버가 아닌 기기 내부(온디바이스)에서 머신러닝 연산을 수행하여 인터넷 연결 없이 동작합니다. [1, 2, 3, 4, 5]

더 자세한 기본 개념과 가이드는 [TensorFlow Lite](https://www.tensorflow.org/lite/guide?hl=ko) 공식 안내서를 참고할 수 있으며, 개념 정리에 대해서는 [TFLite TensorFlow Lite 개념 - 꾸준희 - 티스토리](https://eehoeskrap.tistory.com/678) 및 [임베디드 딥러닝 Tensorflow Lite - 오라즈 - 티스토리](https://oragg.tistory.com/51) 블로그에서 확인할 수 있습니다. 또한 구현 예시는 [텐서플로우 라이트 - velog](https://velog.io/@kingdo/%ED%85%90%EC%84%9C%ED%94%8C%EB%A1%9C%EC%9A%B0-%EB%9D%BC%EC%9D%B4%ED%8A%B8)와 [TensorFlow Lite (Android)를 사용하여 커스텀 객체 감지 모델 빌드 및 배포](https://developers.google.com/codelabs/tflite-object-detection-android?hl=ko) 코랩 문서를 살펴보면 도움이 됩니다. [6, 7, 8]

## 주요 특징

- 가벼운 용량: 파일 크기가 작아서 저장 공간이 부족한 기기에도 쉽게 넣을 수 있습니다.
- 빠른 속도: 기기 안에서 연산할 때 지연 시간이 짧고 반응이 빠릅니다.
- 낮은 전력 소모: 배터리를 아끼면서 효율적으로 인공지능 기능을 쓸 수 있습니다.
- 인터넷 불필요: 네트워크 연결 없이 실시간으로 작동합니다. [1, 5, 9, 10, 11]

## 주요 기능

- 모델 변환: 무거운 인공지능 모델을 작고 가벼운 `.tflite` 형식으로 바꿉니다.
- 양자화 지원: 정확도를 거의 유지하면서 모델 크기와 계산량을 크게 줄입니다.
- 다양한 기기 지원: 안드로이드, 아이폰, 라즈베리 파이 같은 다양한 곳에서 쓸 수 있습니다. [2, 4, 5, 7, 8, 10, 11]

TensorFlow Lite를 적용해보고 싶은 구체적인 플랫폼(안드로이드, iOS, 임베디드 등)이나 사용하려는 AI 모델 종류(이미지 인식, 자연어 처리 등)가 있으신가요? 말씀해주시면 알맞은 변환 및 적용 방법을 안내해 드릴게요.

  

[1] [https://www.tensorflow.org](https://www.tensorflow.org/lite/guide?hl=ko)

[2] [https://wikidocs.net](https://wikidocs.net/336086)

[3] [https://developers.googleblog.com](https://developers.googleblog.com/ko/tensorflow-lite-is-now-litert/)

[4] [https://developers.googleblog.com](https://translate.google.com/translate?u=https://developers.googleblog.com/en/tensorflow-lite-is-now-litert/&hl=ko&sl=en&tl=ko&client=sge)

[5] [https://blog.roboflow.com](https://translate.google.com/translate?u=https://blog.roboflow.com/what-is-tensorflow-lite/&hl=ko&sl=en&tl=ko&client=sge)

[6] [https://eehoeskrap.tistory.com](https://eehoeskrap.tistory.com/678)

[7] [https://oragg.tistory.com](https://oragg.tistory.com/51)

[8] [https://developers.google.com](https://developers.google.com/codelabs/tflite-object-detection-android?hl=ko)

[9] [https://www.quora.com](https://translate.google.com/translate?u=https://www.quora.com/What-is-the-difference-between-TensorFlow-and-TensorFlow-lite&hl=ko&sl=en&tl=ko&client=sge)

[10] [https://www.geeksforgeeks.org](https://translate.google.com/translate?u=https://www.geeksforgeeks.org/deep-learning/introduction-to-tensorflow-lite/&hl=ko&sl=en&tl=ko&client=sge)

[11] [https://medium.com](https://medium.com/@enerzai/tflite-%EB%84%88-t%EC%95%BC-f%EC%95%BC-386badc2c574)