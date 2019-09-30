---
tags : ReactNative
title : RN - react-native-root-import 를 통한 절대경로 import 구현
---

## 개요
리액트 네이티브 개발 시 아주 빈번하게 각 컴포넌트에서 다른 컴포넌트 및 여러 이미지 파일을 import 해서 사용한다. 이때, 다른 경로에 있는 js, img ... 등의 파일에 접근하기 위해서는 일반적으로 `../../` 와 같이 상대경로를 통해 접근하게 된다. 그런데 상대경로가 아닌 `~/components/myIcons/ExpIcon.js`와 같이 절대경로 개념으로 접근하기 위해서는 별도의 라이브러리 사용 및 환경설정이 필요하다. 대표적인 라이브러리 `react-native-root-import`를 사용하여 root 절대경로 import 기능을 구현해보자.

## RN 디렉터리 구조
```
|-- android/
|-- ios/
|-- src/
|   |-- 
|   |-- components/
|	|-- |--	myIcons/
|	|--	|	|-- ExpIcon.js
|   |   |-- card.js
|   |-- images/
|	|--	|	|-- expIcon.png
|   |-- screens/
|   |   |-- index.js
|   |   |-- LoginScreen.js
|   |   |-- HomeScreen.js
|   |-- utils/
|	|   |-- exCalculator.js
|	|	|-- myMoment.js
|-- index.js
```

## 상대경로 접근
ExpIcon.js 파일 내에서 expIcon.png 이미지를 import 하고 싶을 경우                 
`import expIcon from '../../images/expIcon.png'`             
상대경로로 import 하면 위처럼 됨.

## 문제점
`../../` 와 같이 상대경로의 depth가 깊어질 수록 `../../../../` 처럼 다소 복잡한 코드가 형성되며, 폴더 구조를 눈으로 따라가며 depth 만큼 `../`를 추가하게 됨으로써 코드 오류의 소지가 높아지는 문제가 발생.

## 해결방안
`~/images/expIcon.png` 와 같이 src 폴더를 root 로 세팅하여 `~/` 가 src 폴더를 지칭하게끔 세팅할 경우, 상대경로 접근법의 문제점을 해결할 수 있음.             

## react-native-root-import 설치
``` bash
yarn add babel-plugin-root-import --dev
```

`주의` : 하위의 환경설정은 `react-native init [project]` 를 통하여 생성된 프로젝트를 대상으로 함.
`expo 프로젝트는 별도의 서칭이 필요함`

### babel.config.js 파일을 통한 환경 설정
`react-native init` 을 통해 프로젝트를 생성한 후 별도의 바벨 설정을 한 적이 없다면            
일반적으로는 `babel.config.js` 파일을 통한 환경설정만 해당할 것이다.           
단, `babel-preset-react-native` 와 같은 라이브러리를 설치하여 .babelrc 파일을 통해 바벨 설정을 커스터마이징 하고 있는 프로젝트라면      
하위의 .babelrc 파일을 통한 환경 설정을 참조할 것

```javascript
// babel.config.js 파일
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
  // plugins 추가 하면 됨
  plugins: [
  	[
  		'babel-plugin-root-import',
  		{
		    rootPathPrefix: '~', // root 지시자를 ~ 로 설정
		    rootPathSuffix: 'src', // src 폴더를 root 폴더로 설정
	    }
	]
  ],
};
```

### .babelrc 파일을 통한 환경 설정

```javascript
// .babelrc 파일
{
  "plugins": [
  	[
  		'babel-plugin-root-import',
  		{
		    rootPathPrefix: '~', // root 지시자를 ~ 로 설정
		    rootPathSuffix: 'src', // src 폴더를 root 폴더로 설정
	    }
	]
  ]
}
```