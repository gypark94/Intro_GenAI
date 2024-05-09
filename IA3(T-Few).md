### 논문제목: Few-Shot Parameter-Efficient Fine-Tuning is Better and Cheaper than In-Context Learning

LoRA와 비슷한 방법론인데, 더 경제적이 성능이 좋다.

간단한 multiplication으로 rescale하면 적은 파라미터로 튜닝이 가능하다.

#### In-context Learning(ICL)
- GPT-3 : Language models are few-shot learners
- Input-target pair에 대한 computational cost가 높다.
- Fine-tuning에 비해 성능이 떨어진다.
- Template에 따라 결과 변동성이 크다.

#### Few-Shot Parameter-Efficient Fine-Tuning
- In-context learning시 사용하는 소량의 데이터 (20~70개의 examples)
- In-context learning보다 훨씬 좋은 성능
- Full fine-tuning 보다 훨씬 적은 training cost

