---
title: "Unity - 선 샤프트(Sun shafts) / 볼류메트릭 라이팅(Volumetric Lighting) 효과 이용하기"
date: 2018-10-28 17:42:30 +0900
categories: Unity
tags: Unity StandardAssets VolumetricFog VolumetricLighting
comments: true
pinned: true
---
![VolumetricLighting]({{ site.url }}/assets/images/Unity-SunShafts/7.png) 
## Intro
영화, 뮤직비디오, 게임 등 영상 매체를 보다 보면 창문이나 사물 틈으로 멋들어지게 빛이 산란하는 모습을 볼 수 있다. 이러한 빛의 모습을 한국어로는 [틈새빛살](https://ko.wikipedia.org/wiki/%ED%8B%88%EC%83%88%EB%B9%9B%EC%82%B4), 영어로는 [Crepuscular rays](https://en.wikipedia.org/wiki/Crepuscular_rays), `sunbeams`, `sun rays`, `god rays` 등으로 불린다.  
유니티에서는 해당 효과를 `선 샤프트(Sun Shafts)`라 부르고 있고, 유니티 테크놀로지 공식 에셋을 통해 사용할 수 있다.

## Sun Shafts 설치 및 활용
![Effects5.5_0]({{ site.url }}/assets/images/Unity-SunShafts/0.jpg)  
원래는 [스탠다드 에셋(Standard Assets)](https://assetstore.unity.com/packages/essentials/asset-packs/standard-assets-32351)의 포함된 `ImageEffects`의 요소로 존재했었지만 지금은 `ImageEffects`가 [Legacy Image Effects](https://assetstore.unity.com/packages/essentials/legacy-image-effects-83913)라는 이름으로 별도의 에셋으로 분리되었다. 따라서 `Sun Shafts`를 이용하려면 에셋스토어에서 `Legacy Image Effects`를 내려받으면 된다. 이름에 접두로 `Legacy`가 붙은 것과 같이 이전에 주력으로 사용되었지만, 유니티에서는 5.5버전 이후부터 `Legacy Image Effects`가 아닌 `Post Processing Stack`를 이용하길 추천하고 있다. 이에 대한 설명은 이후에 더 설명하겠다.

`Legacy Image Effects`를 정상적으로 임포트에 성공했으면, 선 샤프트 기능을 적용할 카메라를 선택한 다음 메뉴 바의 `Component` - `Image Effects` - `Rendering` - `Sun Shafts`를 선택한다. 그러면 카메라 오브젝트의 `Inspector` 창에 `SunShafts (Script)`가 추가된 것을 확인할 수 있다.

![Effects5.5_1]({{ site.url }}/assets/images/Unity-SunShafts/1.png)  

여러 가지 수치를 조정할 수 있다. 간략히 설명하면 다음과 같다.
- `Resolution`: `Low`, `Normal`, `High` 중에서 고를 수 있다. 선 샤프트의 정밀도, 세밀도와 같이 생각하면 된다.
- `Shafts caster`: 말 그대로 샤프트 효과를 줄 광원이다. 하나만 선택 가능하다. 나는 Scene에서 기존에 사용하던 `Directional light`를 지정했다.
- `Shafts color`: 선 샤프트로 나타나는 흩나리는 빛의 색상을 고른다. 기본은 흰색이다.

### Sun Shafts 적용 결과

#### `Resolution`을 `Low`로 한 경우
![Effects5.5_1]({{ site.url }}/assets/images/Unity-SunShafts/2.png) 

#### `Resolution`을 `High`로 한 경우
![Effects5.5_1]({{ site.url }}/assets/images/Unity-SunShafts/3.png) 

#### `Shafts color`를 노란색 계열로 한 경우
![Effects5.5_1]({{ site.url }}/assets/images/Unity-SunShafts/4.png) 

#### 적용 예
<iframe width="560" height="315" src="https://www.youtube.com/embed/WNO95izzJKw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

#### `Center on Main Camera`를 선택한 경우
<iframe width="560" height="315" src="https://www.youtube.com/embed/YehU4AQ-VCM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### 유니티 5.5버전 이상 버전일 경우.
스탠다드 에셋을 내려받고 `ImageEffects`를 보면 README에서 유니티 5.5버전 이상에서는 [Post Processing Stack](https://assetstore.unity.com/packages/essentials/post-processing-stack-83912) 사용을 추천하고 있다. 하지만 선 샤프트는 해당 에셋에서 찾을 수 없다. 포스트 프로세싱 스택의 [깃허브 이슈](https://github.com/Unity-Technologies/PostProcessing/issues/129#issuecomment-292214260)에 따르면 예정은 있지만 아직 추가되지 않았다는 것 같다. 그래서 우리가 할 수 있는 방법은, 5.5미만 버전과 같이 `Legacy Image Effects`를 활용하거나 유사한 기능을 하는 `VolumetricLighting`을 이용하는 방법이 있다.


## VolumetricLighting
`Volumetric Lighting` 기능을 하는 에셋은 에셋 스토어에 정말 많이 있지만, 그중 유니티 공식 `VolumetricLighting`을 사용해보겠다. 유니티 공식 `VolumetricLighting`은 [Unity Adam Demo](https://unity3d.com/pages/adam)를 위해 만들었다. 에셋스토어 올라와 있지 않고 [깃허브 레포](https://github.com/Unity-Technologies/VolumetricLighting)에서 찾을 수 있다. MIT 라이센스로 저작권 문제없이 사용할 수 있다.

1. `git clone` 혹은 `Download ZIP`으로 내려받고, 자신의 유니티 프로젝트의 `Assets` 디렉토리 밑에 구분이 갈 수 있도록(자신의 프로젝트 파일과 혼동되지 않도록) 내려받은 `VolumetricLighting`의 `Assets` 디렉토리 내부 요소들을 Import 한다.
2. 효과를 적용할 카메라에 `VolumetricFog.cs`를 추가한다.
3. Scene에 `LightManagerFogLights`를 오브젝트로 추가한다.
4. 마찬가지로 Scene에 `AreaLight`를 오브젝트로 추가하고, 하위 요소로 `Directional Light`를 추가한다.  
    (꼭 하위 요소로 둘 필요는 없지만, 하위 요소로 두면 광원 관리에 편하다.)
5. Scene의 Lighting 설정에 Fog가 켜져 있을 경우 체크 해제한다.  
    (`Window` - `Lighting` - `Settings`를 누를 경우 설정 창이 나타난다. 그곳의 가장 하단 `Other Settings`에 있다.)

![VolumetricLighting]({{ site.url }}/assets/images/Unity-SunShafts/5.png) 

완성되면 이런 형태를 갖출 것이다. 그다음으로 할 일은 광원 위치 배치와 수치를 조정하는 일이다.  
나는 나무 사이로 빛이 산란하는 효과를 만들기 위해 상단의 스크린샷과 같이 캐릭터의 옆에서 카메라가 바라보고 있고, 나무 오브젝트로 이루어진 숲 뒤쪽에서 캐릭터(카메라) 방향으로 광원을 만들었다.

### VolumetricFog
카메라에서 공간감을 줄(빛을 산란시킬) Fog의 범위와 수치를 설정한다.
- `Size`: `Near Clip`과 `Far Clip Max`를 설정할 경우, 위의 스크린샷 처럼 범위가 그려질 것이다(노락색 선으로 이루어져 있다). 이 범위와 `AreaLight`에서 설정할 범위가 겹치는 부분에서 빛의 산란이 연산된다.
- `Fog Density`: 이름 그대로 Fog의 밀도를 설정한다. 글로벌 밀도부터 노이즈까지 다양하게 설정할 수 있는데, 수치를 시작부터 높게 잡기보다 0부터 조금씩 높여가며 자신이 원하는 느낌을 찾는 것을 추천한다.

![VolumetricLighting]({{ site.url }}/assets/images/Unity-SunShafts/6.png) 
### AreaLight
마찬가지로 여기서는 산란할 빛의 범위와 강도, 그리고 색상을 설정할 수 있다. 앞서 `VolumetricFog`에서 설명한 것과 같이 범위와 `Directional Light`의 방향을 잡아주면 빛이 흩날리는 효과를 낼 수 있다. 하단의 `Fog Light`에서도 조정할 수 있다.

![VolumetricLighting]({{ site.url }}/assets/images/Unity-SunShafts/7.png) 

## Outro
만약 Windows PC만 지원해도 상관없다면 에셋스토어에서 [Aura Volumetric Lighting](https://assetstore.unity.com/packages/vfx/shaders/aura-volumetric-lighting-111664)를 받아 이용해보는 것도 나쁘지 않다. 무료임에도 불구하고 꽤 멋들어진 빛의 산란을 만들어준다. 다만, DirectX를 활용하여 동작하는 관계로 MacOS나 모바일에서 동작하지 않는다. 또한 처음 임포트를 할 때 컴퓨터 사양에 따라서 20분-1시간 정도 걸릴 수 있으니 차분히 기다리길 추천한다.


## Reference
- [Wikipedia: Volumetric lighting](https://en.wikipedia.org/wiki/Volumetric_lighting)
- [Unity Docs: 스탠다드 에셋](https://docs.unity3d.com/kr/2017.4/Manual/HOWTO-InstallStandardAssets.html)
- [Unity Docs: 선 샤프트](https://docs.unity3d.com/kr/530/Manual/script-SunShafts.html)
- [Github Unity-Technologies/VolumetricLightning](https://github.com/Unity-Technologies/VolumetricLighting)
- [Unity Adam Demo](https://unity3d.com/pages/adam)
- [Assetstore - Aura Volumetric Lighting](https://assetstore.unity.com/packages/vfx/shaders/aura-volumetric-lighting-111664)