# BERT 질문 응답기 통합하기

Task Library `BertQuestionAnswerer` API는 Bert 모델을 로드하고 주어진 문구의 내용을 바탕으로 질문에 답합니다. 자세한 내용은 <a href="../../examples/bert_qa/overview">여기</a>에서 질문-답변 모델에 대한 문서를 참조하세요.

## BertQuestionAnswerer API의 주요 특징

- 두 개의 텍스트 입력을 질문 및 컨텍스트로 받아서 가능한 답변 목록을 출력합니다.

- 입력 텍스트에서 그래프 외 Wordpiece 또는 Sentencepiece 토큰화를 수행합니다.

## 지원되는 BertQuestionAnswerer 모델

다음 모델은 `BertNLClassifier` API와 호환됩니다.

- [질문 응답기를 위한 TensorFlow Lite Model Maker](https://www.tensorflow.org/lite/models/modify/model_maker/question_answer)에서 만든 모델

- [TensorFlow Hub에서 사전 훈련된 BERT 모델](https://tfhub.dev/tensorflow/collections/lite/task-library/bert-question-answerer/1)

- [모델 호환성 요구 사항](#model-compatibility-requirements)을 충족하는 사용자 정의 모델

## Java에서 추론 실행하기

### 1단계: Gradle 종속성 및 기타 설정 가져오기

`.tflite` 모델 파일을 모델이 실행될 Android 모듈의 assets 디렉토리에 복사합니다. 파일을 압축하지 않도록 지정하고 TensorFlow Lite 라이브러리를 모듈의 `build.gradle` 파일에 추가합니다.

```java
android {
    // Other settings

    // Specify tflite file should not be compressed for the app apk
    aaptOptions {
        noCompress "tflite"
    }

}

dependencies {
    // Other dependencies

    // Import the Task Text Library dependency (NNAPI is included)
    implementation 'org.tensorflow:tensorflow-lite-task-text:0.4.4'
}
```

참고: Android Gradle 플러그인 버전 4.1부터는 .tflite가 기본적으로 noCompress 목록에 추가되며 위의 aaptOptions는 더 이상 필요하지 않습니다.

### 2단계: API를 사용하여 추론 실행하기

```java
// Initialization
BertQuestionAnswererOptions options =
    BertQuestionAnswererOptions.builder()
        .setBaseOptions(BaseOptions.builder().setNumThreads(4).build())
        .build();
BertQuestionAnswerer answerer =
    BertQuestionAnswerer.createFromFileAndOptions(
        androidContext, modelFile, options);

// Run inference
List<QaAnswer> answers = answerer.answer(contextOfTheQuestion, questionToAsk);
```

자세한 내용은 [소스 코드](https://github.com/tensorflow/tflite-support/blob/master/tensorflow_lite_support/java/src/java/org/tensorflow/lite/task/text/qa/BertQuestionAnswerer.java)를 참조하세요.

## Swift에서 추론 실행하기

### 1단계: CocoaPods 가져오기

Podfile에 TensorFlowLiteTaskText 포드를 추가합니다.

```
target 'MySwiftAppWithTaskAPI' do
  use_frameworks!
  pod 'TensorFlowLiteTaskText', '~> 0.4.4'
end
```

### 2단계: API를 사용하여 추론 실행하기

```swift
// Initialization
let mobileBertAnswerer = TFLBertQuestionAnswerer.questionAnswerer(
      modelPath: mobileBertModelPath)

// Run inference
let answers = mobileBertAnswerer.answer(
      context: context, question: question)
```

자세한 내용은 [소스 코드](https://github.com/tensorflow/tflite-support/blob/master/tensorflow_lite_support/ios/task/text/qa/Sources/TFLBertQuestionAnswerer.h)를 참조하세요.

## C++에서 추론 실행하기

```c++
// Initialization
BertQuestionAnswererOptions options;
options.mutable_base_options()->mutable_model_file()->set_file_name(model_path);
std::unique_ptr<BertQuestionAnswerer> answerer = BertQuestionAnswerer::CreateFromOptions(options).value();

// Run inference with your inputs, `context_of_question` and `question_to_ask`.
std::vector<QaAnswer> positive_results = answerer->Answer(context_of_question, question_to_ask);
```

자세한 내용은 [소스 코드](https://github.com/tensorflow/tflite-support/blob/master/tensorflow_lite_support/cc/task/text/bert_question_answerer.h)를 참조하세요.

## Python에서 추론 실행하기

### 1단계: pip 패키지 설치하기

```
pip install tflite-support
```

### 2단계: 모델 사용하기

```python
# Imports
from tflite_support.task import text

# Initialization
answerer = text.BertQuestionAnswerer.create_from_file(model_path)

# Run inference
bert_qa_result = answerer.answer(context, question)
```

`BertQuestionAnswerer` 구성에 대한 추가 옵션은 [소스 코드](https://github.com/tensorflow/tflite-support/blob/master/tensorflow_lite_support/python/task/text/bert_question_answerer.py)를 참조하세요.

## 예제 결과

다음은 [ALBERT 모델](https://tfhub.dev/tensorflow/lite-model/albert_lite_base/squadv1/1)의 답변 결과를 보여주는 예입니다.

컨텍스트: "아마존 열대 우림, 또는 영어로 Amazonia로도 알려진 아마존 정글은 남아메리카 아마존 분지 대부분을 덮고있는 아마존 생물 군계의 축축한 활엽 열대 우림입니다. 이 분지는 7,000,000km2(2,700,000평방마일 ), 그중 5,500,000km2(2,100,000평방마일)이 열대 우림으로 덮여 있습니다. 이 지역에는 9개국에 속한 영토가 포함됩니다."

질문: "Where is Amazon rainforest?"

답변:

```
answer[0]:  'South America.'
logit: 1.84847, start_index: 39, end_index: 40
answer[1]:  'most of the Amazon basin of South America.'
logit: 1.2921, start_index: 34, end_index: 40
answer[2]:  'the Amazon basin of South America.'
logit: -0.0959535, start_index: 36, end_index: 40
answer[3]:  'the Amazon biome that covers most of the Amazon basin of South America.'
logit: -0.498558, start_index: 28, end_index: 40
answer[4]:  'Amazon basin of South America.'
logit: -0.774266, start_index: 37, end_index: 40
```

자체 모델 및 테스트 데이터로 간단한 [BertQuestionAnswerer용 CLI 데모 도구](https://github.com/tensorflow/tflite-support/blob/master/tensorflow_lite_support/examples/task/text/desktop/README.md#bert-question-answerer)를 시도해 보세요.

## 모델 호환성 요구 사항

`BertQuestionAnswerer` API는 필수 [TFLite 모델 메타데이터](../../models/convert/metadata)가 있는 TFLite 모델을 예상합니다.

메타 데이터는 다음 요구 사항을 충족해야 합니다.

- Wordpiece/Sentencepiece Tokenizer를 위한 `input_process_units`

- Tokenizer의 출력을 위한 이름이 "ids", "mask" 및 "segment_ids"인 3개의 입력 텐서

- 컨텍스트에서 답변의 상대적 위치를 나타내는 이름이 "end_logits"및 "start_logits"인 2개의 출력 텐서
