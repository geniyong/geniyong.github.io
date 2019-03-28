# CKEditor 5 빌드에 대한 이해     

CKEditor 5 빌드는 설치하자 마자 사용할 수 있는 상태 즉, ready to use 형태로 빌드 되어 있음

그래서 빠른 시작으로 해보기엔 npm 으로 ckeditor 를 설치만 하면 바로 에디터를 웹에 띄울 수 있었음

그런데 빌드에 포함된 구성으로 일부 플러그인 들과 기본 세팅들이 있지만, 완전한 구성은 아니었음

예를 들면, text 의 색을 바꾸는 하이라이팅 버튼이 없어서 docu 를 참조하면서, 

highlight plugin 설치 및 import 를 진행했지만, Webpack 에서 svg 를 가져오지 못하여 에러가 발생함.

`ERROR TypeError` : Cannot read property 'getAttribute' of null

![error.png](https://postfiles.pstatic.net/MjAxOTAzMTJfMjk0/MDAxNTUyMzg0MjA4NDQy.SxxK51iD_vwDU7NFk40yYV1KDZ1axsuGuMEch_tbkxIg.GgUnlKgXrauKAC0y9TKZMZdw8RSWtGAgT_IjgBSZZY4g.PNG.inforsec0201/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2019-03-12_%EC%98%A4%ED%9B%84_6.49.39.png?type=w966)

그래서 다시 documentation 을 천천히 읽어 본 결과,

플러그인을 추가하기 위해서는 에디터를 다시 빌드하는 작업이 필요하다고 함

​

< 에디터를 다시 빌드 > 는 내 리액트 프로젝트를 빌드하는 개념이 아니라 내가 받은 CKEditor 라이브러리를 다시 빌드해야 함을 의미 함.

​

이렇게 다시 빌드 하는 작업 - 단, custom 된 소스코드를 빌드하는 작업 은 크게 두 방법으로 가능하다고 함

하나는, CKEditor 의 소스코드를 github 에서 cloning 한 뒤 코드를 고치고 재 빌드 하는 방법

( ref : https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/installing-plugins.html )

​

하나는, 플러그인을 수정할 때마다 에디터의 재빌드 과정 없이 나의 리액트 어플리케이션 코드와 유기적으로 통합시키는 방법

( ref : https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/advanced-setup.html )

​

본 포스팅에서는 두번째 방법을 진행함.

왜냐면 첫번째 방법은 내 프로젝트의 설정은 건드릴 필요 없이 에디터만 재 빌드 하면 되는데,

두번째 방법은 내 프로젝트의 webpack 설정을 건드려야 하기 때문에 webpack 설정에 대한 정리가 필요.

# 플러그인 사용을 위한 환경 설정

## 설정 파일 추출

일반적으로 많은 개발자들이 react 프로젝트 시작 시,

react 의 보일러플레이트에 해당하는 create-react-app 을 사용하여 시작함

create-react-app 을 통해서 리액트 프로젝트를 생성 시 기본적으로 webpack 설정이 잡혀있는 상태로 출발함

우선적으로, 이렇게 기본적으로 잡혀있는 설정사항을 추출해야함

```bash
npm run eject
```

documentation 에서 말하기를 webpack config 파일을 추출하고 난 뒤,

UglifyJsPlugin 에 관한 선택사항을 설정해야 했지만, webpack ver 4.0 부터는 기본적으로 제공해주기 때문에 할 필요가 없다고 함.

최근에 리액트 앱을 생성하면 webpack@4 버전을 설치하기 때문에 생략하기로 함

## WebPack 설정 변경하기 ( dev, prod 파일 모두 변경 필요 )

npm run eject 명령어를 통해 webpack 설정을 추출하고 나면

다음 경로에 설정파일이 추출 됨

```
<project_root>/config/webpack.config.dev.js
<project_root>/config/webpack.config.prod.js
```

또는

```
<project_root>/config/webpackDevServer.config.js  // dev
<project_root>/config/webpack.config.js  // prod
```

~~반드시 아래의 설정을 두 파일 모두에 적용해야 함~~
아래의 설정을 webpack.config.js (배포용 설정파일) 에만 적용하면 됨

### 1. PostCSS 를 위한 설정이 담긴 style object import

```javascript
const { styles } = require( '@ckeditor/ckeditor5-dev-utils' );
```

### 2. SVG 와 CSS 파일 로더 설정을 위해 module.rules 안에 다음 코드 삽입
``` javascript
// module.rules 아래에
/*
    module: {
      strictExportPresence: true,
      rules: [
       ... 여기에 삽입 ...
      ]
*/
{
  test: /ckeditor5-[^/\\]+[/\\]theme[/\\]icons[/\\][^/\\]+\.svg$/,
  use: [ 'raw-loader' ]
},
{
  test: /ckeditor5-[^/\\]+[/\\]theme[/\\].+\.css/,
  use: [
    {
      loader: 'style-loader',
      options: {
        singleton: true
      }
    },
    {
      loader: 'postcss-loader',
      options: styles.getPostCssConfig( {
        themeImporter: {
          themePath: require.resolve( '@ckeditor/ckeditor5-theme-lark' )
        },
        minify: true
      } )
    }
  ]
},
```

### 3. 리액트 프로젝트의 웹에서는 css 파일 적용 안되게 배제 시키기

주의! 아래 코드는 기존의 코드에서 수정해야 하는 부분임

/\.css$/ 부분이 cssRegex 변수로 설정 되어 있을 수 있음


``` javascript
{
  test: /\.css$/,
  exclude: /ckeditor5-[^/\\]+[/\\]theme[/\\].+\.css/,
  // (...)
}

```

### 4.  CKEditor 5 의 SVG 와 CSS 파일들은 파일로더로 부터 배제시키기

기존에 추가 되어 있던 파일 로더에 의해 관리 되기 때문

아래의 코드는 기존의 loader 코드 위에 덮어쓰기

``` javascript
{
  loader: require.resolve('file-loader'),
  // Exclude `js` files to keep the "css" loader working as it injects
  // its runtime that would otherwise be processed through the "file" loader.
  // Also exclude `html` and `json` extensions so they get processed
  // by webpack's internal loaders.
  exclude: [
    /\.(js|jsx|mjs)$/,
    /\.html$/,
    /\.json$/,
    /ckeditor5-[^/\\]+[/\\]theme[/\\]icons[/\\][^/\\]+\.svg$/,
    /ckeditor5-[^/\\]+[/\\]theme[/\\].+\.css/
  ],
  options: {
    name: 'static/media/[name].[hash:8].[ext]'
  }
}

```

# 커스텀 플러그인 연동

webpack 설정은 끝!

이제 raw-loader 및 CKEditor 5 테마와 개발 유틸 라이브러리를 설치 한 뒤

실제 커스텀 플러그인을 설치 및 추가하여 에러없이 잘 되는지 확인 해 볼 것

## 라이브러리 설치
``` bash
npm install --save-dev raw-loader @ckeditor/ckeditor5-theme-lark @ckeditor/ckeditor5-dev-utils
```

## 플러그인 설치 ( 역슬래시  제거 후 선택해서 설치 하면 됨 )

``` bash
npm install --save \
    @ckeditor/ckeditor5-react \
    @ckeditor/ckeditor5-editor-classic \
    @ckeditor/ckeditor5-essentials \
    @ckeditor/ckeditor5-basic-styles \
    @ckeditor/ckeditor5-heading \
    @ckeditor/ckeditor5-paragraph
```

## CKEditor 리액트 렌더

``` javascript
import React, { Component } from 'react';
import CKEditor from '@ckeditor/ckeditor5-react';

import ClassicEditor from '@ckeditor/ckeditor5-editor-classic/src/classiceditor';
import Essentials from '@ckeditor/ckeditor5-essentials/src/essentials';
import Paragraph from '@ckeditor/ckeditor5-paragraph/src/paragraph';
import Bold from '@ckeditor/ckeditor5-basic-styles/src/bold';
import Italic from '@ckeditor/ckeditor5-basic-styles/src/italic';
import Heading from '@ckeditor/ckeditor5-heading/src/heading';

class App extends Component {
    render() {
        return (
            <div className="App">
                <h2>Using CKEditor 5 Framework in React</h2>
                <CKEditor
                    onInit={ editor => console.log( 'Editor is ready to use!', editor ) }
                    onChange={ ( event, editor ) => console.log( { event, editor } ) }
                    config={ {
                        plugins: [ Essentials, Paragraph, Bold, Italic, Heading ],
                        toolbar: [ 'heading', '|', 'bold', 'italic', '|', 'undo', 'redo', ]
                    } }
                    editor={ ClassicEditor }
                    data="<p>Hello from CKEditor 5!</p>"
                />
            </div>
        );
    }
}

export default App;
```

# Note

커스텀 빌드를 사용하는 순간 부터는 기존의 제공되던 플러그인들도 라이브러리 설치 후 import 하여 사용하여야 함.

예를 들면, Essentials 플러그인 같은 경우 quick start 에서는 따로 설치하지 않아도 또한 플러그인에 따로 추가하지 않았어도 이미 포함되어있던 상태였지만, 커스텀 빌드를 하였기 때문에 따로 설치 및 추가해주지 않으면 안됨

​
내가 사용하는 플러그인들.. 모두 포함한 소스코드

``` js
import React from 'react';
import './index.css'

import CKEditor from '@ckeditor/ckeditor5-react';
import ClassicEditor from '@ckeditor/ckeditor5-build-classic';
import InlineEditor from '@ckeditor/ckeditor5-build-inline';
import Highlight from '@ckeditor/ckeditor5-highlight/src/highlight';
import Essentials from '@ckeditor/ckeditor5-essentials/src/essentials';
import Paragraph from '@ckeditor/ckeditor5-paragraph/src/paragraph';
import Bold from '@ckeditor/ckeditor5-basic-styles/src/bold';
import Italic from '@ckeditor/ckeditor5-basic-styles/src/italic';
import Heading from '@ckeditor/ckeditor5-heading/src/heading';
import UploadAdapter from '@ckeditor/ckeditor5-adapter-ckfinder/src/uploadadapter';
import Autoformat from '@ckeditor/ckeditor5-autoformat/src/autoformat';
import BlockQuote from '@ckeditor/ckeditor5-block-quote/src/blockquote';
import EasyImage from '@ckeditor/ckeditor5-easy-image/src/easyimage';
import Image from '@ckeditor/ckeditor5-image/src/image';
import ImageCaption from '@ckeditor/ckeditor5-image/src/imagecaption';
import ImageStyle from '@ckeditor/ckeditor5-image/src/imagestyle';
import ImageToolbar from '@ckeditor/ckeditor5-image/src/imagetoolbar';
import ImageUpload from '@ckeditor/ckeditor5-image/src/imageupload';
import Link from '@ckeditor/ckeditor5-link/src/link';
import List from '@ckeditor/ckeditor5-list/src/list';
import Alignment from '@ckeditor/ckeditor5-alignment/src/alignment'; 
import CKFinder from '@ckeditor/ckeditor5-ckfinder/src/ckfinder';

// Components

export default class MyEditor extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return (
      <div className="editorContainer">
        <h2>Using CKEditor 5 build in React</h2>
        <CKEditor
            editor={ ClassicEditor }
            data="<p>Hello from CKEditor 5!</p>"
            onInit={ editor => {
                // You can store the "editor" and use when it is needed.
            } }
            onChange={ ( event, editor ) => {
                const data = editor.getData();
                console.log( { event, editor, data } );
            } }
            onBlur={ editor => {
                console.log( 'Blur.', editor );
            } }
            onFocus={ editor => {
                console.log( 'Focus.', editor );
            } }
            config = {{
              plugins: [ CKFinder, Highlight, Essentials, Paragraph, Bold, Italic, Heading, UploadAdapter, Autoformat, BlockQuote, 
              EasyImage, Image, ImageCaption, ImageStyle, ImageToolbar, ImageUpload, Link, List, Alignment ],

              highlight: {
                options: [
                  {
                      model: 'redPen',
                      class: 'pen-red',
                      title: 'Red pen',
                      color: 'var(--ck-highlight-pen-red)',
                      type: 'pen'
                  },
                  {
                      model: 'greenPen',
                      class: 'pen-green',
                      title: 'Green pen',
                      color: 'var(--ck-highlight-pen-green)',
                      type: 'pen'
                  },
                  {
                      model: 'yellowMarker',
                      class: 'marker-yellow',
                      title: 'Yellow marker',
                      color: 'var(--ck-highlight-marker-yellow)',
                      type: 'marker'
                  },
                  {
                      model: 'greenMarker',
                      class: 'marker-green',
                      title: 'Green marker',
                      color: 'var(--ck-highlight-marker-green)',
                      type: 'marker'
                  },
                  {
                      model: 'pinkMarker',
                      class: 'marker-pink',
                      title: 'Pink marker',
                      color: 'var(--ck-highlight-marker-pink)',
                      type: 'marker'
                  },
                  {
                      model: 'blueMarker',
                      class: 'marker-blue',
                      title: 'Blue marker',
                      color: 'var(--ck-highlight-marker-blue)',
                      type: 'marker'
                  },
                ]
              },
              toolbar: {
                items:
                [
                  'heading', '|', 
                  'alignment',  
                  'bold', 'italic', 'highlight', 'link', 'bulletedList', 
                  'numberedList', 'imageUpload', 'blockQuote', 'insertTable', 
                  'mediaEmbed', 'undo', 'redo'
                ],

              },
              image: {
                toolbar: [
                    'imageStyle:full',
                    'imageStyle:side',
                    '|',
                    'imageTextAlternative'
                ]
              },
              heading: {
                  options: [
                      { model: 'heading1', view: 'h1', title: '헤더1', class: 'ck-heading_heading1' },
                      { model: 'heading2', view: 'h2', title: '헤더2', class: 'ck-heading_heading2' },
                      { model: 'heading3', view: 'h3', title: '헤더3', class: 'ck-heading_heading3' },
                      { model: 'paragraph', title: '본문', class: 'ck-heading_paragraph' },
                  ]
              },
              ckfinder: {
                uploadUrl: 'http://api.dev.mustrip.io/meetup/upload/files/'
              },
            }}
        />      
      </div>
    );
  }
}

```
