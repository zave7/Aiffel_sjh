### [피어리뷰 보러가기(클릭)](PRT.md)

# 프로젝트 설명
**프로젝트: 프로젝트: CutMix 또는 Mixup 비교실험 하기**


## 데이터

# 프로젝트 진행 과정
1. 

# 목표
1.  CutMix와 MixUp 기법을 ResNet50 분류기에 성공적으로 적용하였는가? [모델 설명](#모델-설명)
- CutMix와 MixUp을 적용한 데이터셋으로 훈련한 각각의 ResNet 모델이 수렴하였다.

2. 다양한 실험을 통해 태스크에 최적인 Augmentation 기법을 찾아내었는가?[실행 결과](#실행 결과)
- 각 Augmentation 기법을 적용하고, 그에 따른 성능 비교 분석 및 문제점을 서술하였음

3. 여러가지 Augmentation 기법을 적용한 결과를 체계적으로 비교분석하였는가? [실행 결과]()

- 기본 Augmentation, CutMix, MixUp이 적용된 결과를 시각화와 함께 체계적으로 분석하였다.

# 파일 설명

* /model 폴더

최대한 코드들을 모듈화 시켜 테스트를 진행해보고 싶었지만 ipynb특성상 2차 import가 힘든거 같아 더이상 진행 X

* [Project_trial.ipynb](Project_trial.ipynb)

코드의 전반적인 실행을 위해 기본 코드들을 생성한 ipynb

* [cutmix_test.ipynb](cutmix_test.ipynb)

Augmentation의 랜덤화 및 다양한 환경의 테스트를 진행한 ipynb

* [mixup_test.ipynb](mixup_test.ipynb)

이전 실험 후 Mixup의 성능 테스트를 진행하기 위해 새로 진행한 테스트 ipynb

# 모델 설명
## CutMix

```{python}
# cutmix
def cutmix(image, label, prob = 1.0, batch_size = 16, img_size = 224):
    mixed_imgs = []
    mixed_labels = []

    for i in range(batch_size):
        image_a = image[i]
        label_a = label[i]
        j = tf.cast(tf.random.uniform([], 0, batch_size), tf.int32)
        image_b = image[j]
        label_b = label[j]
        x_min, y_min, x_max, y_max = get_clip_box(image_a, image_b)
        mixed_img, mixed_label = mixer(image_a, image_b, label_a, label_b, x_min, y_min, x_max, y_max)
        mixed_imgs.append(mixed_img)
        mixed_labels.append(mixed_label)

    mixed_imgs = tf.reshape(tf.stack(mixed_imgs), (batch_size, img_size, img_size, 3))
    mixed_labels = tf.reshape(tf.stack(mixed_labels), (batch_size, num_classes))
    return mixed_imgs, mixed_labels
```

## MixUp

```{python}
# function for mixup
def mixup_2_images(image_a, image_b, label_a, label_b):
    ratio = tf.random.uniform([], 0, 1)

    if len(label_a.shape)==0:
        label_a = tf.one_hot(label_a, num_classes)
    if len(label_b.shape)==0:
        label_b = tf.one_hot(label_b, num_classes)
    mixed_image= (1-ratio)*image_a + ratio*image_b
    mixed_label = (1-ratio)*label_a + ratio*label_b

    return mixed_image, mixed_label

def mixup(image, label, prob=1.0, batch_size=16, img_size=224, num_classes=120):
    mixed_imgs = []
    mixed_labels = []

    for i in range(batch_size):
        image_a = image[i]
        label_a = label[i]
        j = tf.cast(tf.random.uniform([],0,batch_size), tf.int32)
        image_b = image[j]
        label_b = label[j]
        mixed_img, mixed_label = mixup_2_images(image_a, image_b, label_a, label_b)
        mixed_imgs.append(mixed_img)
        mixed_labels.append(mixed_label)

    mixed_imgs = tf.reshape(tf.stack(mixed_imgs), (batch_size, img_size, img_size, 3))
    mixed_labels = tf.reshape(tf.stack(mixed_labels), (batch_size, num_classes))
    return mixed_imgs, mixed_labels
```

## Random Augmentation

```
# augment with randomness
def augment(image, label):
    if random.choice([True, False]):
        image = tf.image.random_flip_left_right(image)
    if random.choice([True, False]):
        image = tf.image.random_flip_up_down(image)
    if random.choice([True, False]):
        brightness_factor = random.uniform(0, 0.2)
        image = tf.image.random_brightness(image, brightness_factor)
    if random.choice([True, False]):
        contrast_lower = random.uniform(0.2, 0.5)
        contrast_upper = random.uniform(contrast_lower, 0.5)
        image = tf.image.random_contrast(image, contrast_lower, contrast_upper)
    if random.choice([True, False]):
        hue_factor = random.uniform(0, 0.2)
        image = tf.image.random_hue(image, hue_factor)
    if random.choice([True, False]):
        saturation_lower = random.uniform(0.2, 0.5)
        saturation_upper = random.uniform(saturation_lower, 0.5)  # Ensure upper is always >= lower
        image = tf.image.random_saturation(image, saturation_lower, saturation_upper)
    image = tf.clip_by_value(image, 0, 1)
    return image, label
```

# 실행 결과

## resnet50 vs aug_resnet50
![fist](img/accuracy1.png)

20에폭으로 테스트를 진행해봤는데 단순 CutMix Augmentation을 취한 모델이 전체적으로 취하지 않은 모델보다 성능 면에서 좋지 못한 결과물을 보이고 있다
-> 다음실험: 여러 Augmentation 기법들을 수행해보고 성능면에서 진척도를 보이는 기법 찾기

## Random Augmentation

![aaug](img/aug.png)

이미지 Augmentation 기법은 위 코드처럼 다양하게 있는데, 이를 모든 이미지에서 처리보다는 모든 부분에 랜덤성을 부여하면 좋지 않을까 라는 새로운 가설

![ran](img/ranaug.png)

이처럼 각 코드의 줄의 실행여부, 수치정도 모두 랜덤성 부여해서 augmentation을 진행하게 수정

## Random Aug & CutMix

![50](img/20epoch.png)

[코드](mixup_test.ipynb)

augmentation, cutmix 적용에 랜덤성을 부여하는 방식으로 진행해 보았는데 augmentation을 넣지 않고 cutmix만 진행한 경우 가장 validation에서 높은 정확도 및 낮은 에러를 보이는 것을 볼 수 있다.

-> 가정 1: augmenation은 모델 성능에 악 영향을 끼친다
-> 가정 2: 특정 augmentation 기법중에 모델 성능에 악영향을 주지 않는 기법도 있을 수 있다

-> 실험 방법: 가정 1,2 모두 확인을 위해서는 cutmix는 제외해 두고 각 augmentation 기법 별 성능 변화추이를 확인하면 체크 가능
[이동](#다음-실험augmentation)

또한 augmentation만 수행한 모델의 경우 시작 성능이 꾀 높게 나오지만 학습에 변동성이 보이지 않아, 더 많은 epoch을 줘야지 augmentation 기법에 대해 제대로 다룰 수 있을것으로 판단

## CutMix vs MixUp

공정한 비교를 위해 전 실험에서 의구심이 많이 들었던 augmentation 기법을 우선 사용하지 않고 CutMix 와 MixUp만을 사용해서 20epoch으로 테스트 진행

![mixs](img/cutmixmixup.png)

[코드](mixup_test.ipynb)

두가지 경우의 수만을 가지고 테스트를 진행해본 결과 크게 어떤 모델이 더 뛰어나다고는 해당 결과만을 가지고는 제대로 확인 불가로 보임

-> 새로운 실험: 대략 50 에폭 수준의 새로운 테스트 진행
[이동](#다음-실험50에폭)

## 다음 실험(augmentation)


## 다음 실험(50에폭)

# 회고

데이터 증강에 대해서 좀더 깊게 다룰 수 있던 시간이었습니다.

## 배운점


## 아쉬운점


## 느낀점


## 궁금한 내용



# 참고자료

* 