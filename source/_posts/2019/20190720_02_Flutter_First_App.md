---
title: "나의 최초 플루터 앱 Part2"
date: 2019-07-20 18:20:00
categories:
- flutter
tags:
- flutter
keywords:
- flutter
autoThumbnailImage: false
thumbnailImagePosition: "bottom"
thumbnailImage: https://flutter.dev/assets/homepage/news-2-599aefd56e8aa903ded69500ef4102cdd8f988dab8d9e4d570de18bdb702ffd4.png
metaAlignment: center
---

최초 Flutter App의 두번째 파트입니다.

<!-- excerpt -->

> 해당 소스는 Flutter 공식 사이트를 참고 했어요.

이번장에서는 전 장에서 만든 무한스크롤을 좀더 기능 개선을 했어요. 여기에서 배울점은 아래와 같아요.
 
1. 자연스럽게 보이는 Flutter 앱을 작성하는 방법.
2. 더 빠른 개발 위한 Hot Reload 사용 방법.
3. Stateful Widget에 대화 형 작업을 추가하는 방법.
4. 두 번째 화면을 만들고 탐색하는 방법.
5. 테마를 사용하여 앱의 모양을 변경하는 방법.

예전 무한 스크롤에서 내가 좋아하는 단어를 즐겨찾기 할 수 있는 기능을 만들어 볼꺼에요. 자 신나게 만들어봐요 :)

만약에 나의 최초 플로터 앱 Part1을 보지 않았으면 보고 오시는게 좋아요, 이번장은 기존 코드에 기능을 붙여 나가는 형태이기 때문이에요. [Part1는 여기](https://agoalofglory.github.io/2019/07/20/2019/20190720_01_Flutter_First_App/)

### 리스트에 아이콘 추가하기

각 행에 하트 아이콘을 추가해봐요.

RandomWordsState에 _saved 변수를 추가합니다. 이 배열은 사용자가 선호 한 단어를 저장합니다. Set을 사용해서 _saved는 중복된 항목을 허용하지 않도록 하려고 합니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class RandomWordsState extends State<RandomWords> {
  final _suggestions = <WordPair>[];
  final _biggerFont = const TextStyle(fontSize: 18.0);
  final Set<WordPair> _saved = Set<WordPair>();   // _saved는 즐겨찾기한 단어의 모음
  ...
}
{% endcodeblock %}

_buildRow함수에서 alreadySaved변수를 만들고 해당변수는 _saved에 인자로 받는 단어의 중복을 체크하는 용도로 사용할꺼에요.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
Widget _buildRow(WordPair pair) {
  final bool alreadySaved = _saved.contains(pair);
  ...
}
{% endcodeblock %}

_buildRow()에서는 ListTile 객체에 하트 모양의 아이콘을 추가하여 즐겨찾기 기능을 넣을꺼에요. 아래 코드처럼 alredySaved 변수를 통해서 ListTile에 하트 아이콘을 넣어봐요.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
Widget _buildRow(WordPair pair) {
// _saved에 인자로 받는 값과 동일한 값이 있는지 체크합니다.
final bool alreadySaved = _saved.contains(pair);
  
  return ListTile(
    title: Text(
      pair.asPascalCase,
      style: _biggerFont,
    ),
    // 중복상황에 따라서 빨간 하트를 표시하거나 그냥 border만 있는 하트를 표시한다.
    trailing: Icon(
      alreadySaved ? Icons.favorite : Icons.favorite_border,
      color: alreadySaved ? Colors.red : null
    ),
  );
}
{% endcodeblock %}

### 상호작용 추가

마음에 드는 단어를 탭해서 해당 단어를 즐겨찾기 모음에 추가하거나 제거하는 기능을 만들꺼에요.

아래와 같이 buildRow함수를 수정해야 합니다. 단어 항목이 이미 추가되었을 경우 즐겨찾기 목록에서 제거하고 즐겨찾기 목록에 존재하지 않는 단어라면 즐겨찾기 목록에 해당 단어를 넣을꺼에요.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
Widget _buildRow(WordPair pair) {
// _saved에 인자로 받는 값과 동일한 값이 있는지 체크합니다.
final bool alreadySaved = _saved.contains(pair);
  
  return ListTile(
    title: Text(
      pair.asPascalCase,
      style: _biggerFont,
    ),
    // 중복상황에 따라서 빨간 하트를 표시하거나 그냥 border만 있는 하트를 표시한다.
    trailing: Icon(
      alreadySaved ? Icons.favorite : Icons.favorite_border,
      color: alreadySaved ? Colors.red : null
    ),
    onTap: () {
      // ListTile을 탭할때 setState()를 호출하여 state가 변경됨을 Flutter에게 알립니다.
      setState(() {
        if(alreadySaved) {
          _saved.remove(pair);
        } else {
          _saved.add(pair);
        }
      });
    }
  );
}
{% endcodeblock %}

setState()함수를 호출하면 State객체에 대한 build()함수가 호출되어 UI가 재 랜더링 됩니다.

### 새로운 화면으로 이동

즐겨 찾기를 표시하는 새로운 페이지를 추가 합니다. Router를 사용해서 홈 경로와 새 페이지 경로를 탐색하는 방법을 배우게 됩니다.

Flutter에서 네비게이터는 앱의 경로가 포함된 스택을 관리합니다. 네비게이터의 스택으로 경로를 push하면 디스플레이가 해당 경로로 업데이트 됩니다. 네비게이터의 스택에서 경로를 pop하면 디스플레이가 이전 경로로 돌아갑니다. 

자 RandomWordsState의 build 메서드에서 AppBar에 List아이콘을 추가합니다. 사용자가 List아이콘을 클릭하면 저장된 즐겨찾기가 표시되는 새페이지 경로가 Navigator에 push되어서 새로운 페이지로 이동을 할 것입니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title : Text('infinite Scroll Word'),
      actions: <Widget>[
        IconButton(icon: Icon(Icons.list), onPressed: _pushSaved),
      ]
    ),
    body : _buildSuggestions()
  );
}
{% endcodeblock %}

List 아이콘을 클릭시 실행할 함수도 만들어 줍니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
void _pushSaved() {
}
{% endcodeblock %}

다음은 Route를 만들고 Navigator의 스택에 푸시하는 코드를 만들 것입니다. 이코드는 새 페이지 경로를 표시하도록 화면을 변경합니다. 새 페이지의 내용은 MaterialPageRoute의 build 함수에 익명 함수로 작성됩니다.

네비게이터의 스택에 경로를 푸시하는 코드는 아래 그림과 같이 Navigator.push를 호출합니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
void _pushSaved() {
  Navigator.of(context).push();
}
{% endcodeblock %}

다음에는 MaterialPageRoute와 그 빌더를 추가합니다. 지금은 ListTile 행을 생성하는 코드를 추가하십시오. 
ListTile의 divideTiles() 메서드는 각 ListTile 사이에 가로 간격 줄을 추가합니다. 가로간격줄이 들어간 리스트는 변환된 마지막 행을 toList()로 받아올수 있습니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}

{% endcodeblock %}









객체지향코드와 기본적인 프로그래밍에 익숙하다면 이번장에서 만드는 Flutter 코드가 이해가 될꺼라 생각이 들어요.

만들어볼 앱은 제한이 없는 스크롤 리스트 입니다. 여기서 배워갈 점은 아래와 같아요.

1. Flutter App을 작성하는 방법.
2. Flutter App의 기본 구조.
3. 검색과 패키지들을 사용해서 기능을 확장하는 방법.
4. 빠른 개발을 Hot reload 사용 방법.
5. Stateful widget을 개발하는 방법.
6. 무한하고 lazy하게 로드된 리스트를 생성하는 방법.

## Flutter App 만들기

새로운 플루터 앱을 만들어 봅니다. 이름은 flutter_first_app 할께요.

이상 없이 생성이 되면 에뮬레이터를 실행시키고 새로 생성된 Flutter app을 기동시키세요.

우하단에 파란색 버튼과 You have pushed the button this many time: 0이라는 글씨가 중앙에 뜬 앱이 보인다면 이상없이 기본 Flutter App이 기동이 된거에요.

### 1. lib/main.dart파일을 아래 코드로 변경해주세요.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Welcome to Flutter'),
        ),
        body: Center(
          child: Text('Hello World'),
        ),
      ),
    );
  }
}
{% endcodeblock %}

Hot Reload기능으로 인해서 코드를 저장시 실행된 애뮬레이터에 해당 코드가 반영됨을 확인 할 수 있어요. Hello World가 중앙에 뜨면 이상없이 적용이 된거에요.

**확인**

- Material App을 만들었습니다. Material은 모바일과 웹에서 시각적인 디자인 언어에요, Flutter는 많은 Material Widget들을 제공해줍니다.
- Main 함수에서 Arrow(=>) 표기법을 사용했어요. 한줄에서 표현이 가능한 함수나 메소드는 Arrow를 사용하세요.
- MyApp은 StatelessWidget을 확장한 Widget이에요. Flutter는 대부분이 Widget입니다. alignment, padding, layout도 Widget이에요.
- Scaffold Widget을 사용했어요. 이건 기본적으로 AppBar, Title 속성 및 Home Screen 영역을 담당하고 자식 Widget Tree를 유지하는 body 속성을 제공합니다.
- Widget의 주요 작업은 build() 함수를 제공하는 거에요. build는 위젯을 어떻게 표시할지에 대한 코드들이 작성이 된답니다.
- 예제에서는 Center Widget을 사용해서 수직, 수평으로 중앙에 Text Widget을 표시해주고 있습니다.

### 2. 외부 패키지 사용하기

english_word라는 외부 패키지를 사용해봐요. english_word는 가장 많이 사용되는 영어단어의 제공과 몇몇의 기능을 제공합니다.

Flutter Pub(https://pub.dev/flutter) 사이트에서 사람들이 올려놓은 외부 패키지를 검색해보세요. english_word를 검색해보세요.

아래처럼 pubspec.yaml에 english_word를 작성합니다.

{% codeblock pubspec.yaml lang:yaml http://underscorejs.org/#compact pubspec.yaml %}
dependencies:
  flutter:
    sdk: flutter
  english_words: ^3.1.0
{% endcodeblock %}

이상없이 추가 되면 'flutter pub get'을 터미널에 입력하거나 Tool에 표시되는 'package get'을 눌러봐요. 이건 선언한 소스를 앱에서 사용할 수 있게 다운받고 설정을 합니다. 이것은 pubspec.lock을 자동으로 생성하며 프로젝트에 끌어온 모든 패키지의 목록을 파일로 저장하고 버전 관리 합니다.

아래와 같이 main.dart에 english_word를 선언 해보죠.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';
{% endcodeblock %}

자 english_word를 사용하여 Hello world를 english_word가 제공해주는 단어로 변경해보죠.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();

    return MaterialApp(
      title: 'Welcome to Flutter',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Welcome to Flutter'),
        ),
        body: Center(
          child: Text(wordPair.asPascalCase),
        ),
      ),
    );
  }
}
{% endcodeblock %}

이상없이 작성하셨으면 Hello world가 랜덤한 Pascal case형태의 영어단어로 변경이 되었을 꺼에요.

코드를 수정하거나, Hot Reload 버튼을 클릭하면 중앙 텍스트가 계속 변경이 됩니다. 이것은 단어를 선택하는 WordPair.random();코드가 build 함수 안에 있기 때문입니다.

MaterialApp에 렌더링이 필요할때나 표시되는 플랫폼이 전환될때 build함수가 호출이 되어서 새로운 단어를 생성하고 적용하고 렌더링을 하게되요.

### 3. Stateful Widget 추가하기

Stateless란 불변의 의미를 지닙니다. Stateless Widget이란 사용되는 데이터의 값을 변경을 할 수 없는 위젯이며, 한번 지정된 값은 코드가 수정되지 않는한 변경을 할수 없음을 의미합니다.

Stateful Widget은 Stateless Widget과 다르게 불변하지 않으며 상황에 따라 사용하는 값을 변경할 수 있음을 의미합니다.

Stateful Widget은 2개의 클래스가 필요한데요, 하나는 StatefulWidget클래스와 State 클래스 입니다. StatefulWidget은 해당 위젯의 수명주기를 관리하는 State 클래스의 인스턴스를 생성하는 역할을 합니다.  

RandomWordsState 클래스를 추가해봅니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class RandomWordsState extends State<RandomWords> {
  // TODO Add build() method
}
{% endcodeblock %}

코드를 보면 RandomWordsState는 RandomWords를 위한 State 클래스를 확장한 것을 알 수 있습니다. 앱의 로직과 상태는 대부분 RandomWords Widget의 상태를 유지합니다.

RandomWords Widget을 추가해봅니다. RandomWords Widget은 RandomWordsState 클래스의 인스턴스를 만드는 외에 다른 작업은 거의 수행하지 않아요.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class RandomWords extends StatefulWidget {
  @override
  RandomWordsState createState() => RandomWordsState();
}

class RandomWordsState extends State<RandomWords> {
  // TODO Add build() method
}
{% endcodeblock %}

이후 IDE는 RandomWordsState클래스에 build메소드를 만들라고 알려줍니다. 아래처럼 build 메소드를 생성하고 랜덤한 Text Widget을 반환하도록 작성합니다. 

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class RandomWords extends StatefulWidget {
  @override
  RandomWordsState createState() => RandomWordsState();
}

class RandomWordsState extends State<RandomWords> {
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();
    return Text(wordPair.asPascalCase);
  }
}
{% endcodeblock %}

이전에 표시를 담당하는 body속성의 Center 안의 child값에 해당 RandomWords위젯을 호출하도록 작성합니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
  
    final wordPair = WordPair.random();
    
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Welcome to Flutter'),
        ),
        body: Center(
          child: RandomWords(),
        ),
      ),
    );
  }
}
{% endcodeblock %}


### 4. 무한 스크롤 리스트 뷰 만들기

이 단계에서는 RandomWordsState를 확장하여 단어 목록을 생성하고 표시를 합니다. 사용자가 스크롤 하면 ListView 위젯에 표시된 목록이 무한대로 커집니다.

ListView의 Builder Factory 생성자를 사용하면 필요할 때 Listview를 Lazy하게 빌드 할 수 있습니다.

단어를 저장하기 위해서 _suggestions라는 WordPair 배열을 선언합니다. 그리고 글꼴의 크기를 크게 하기 위해서 _biggerFont라는 TextStyle 인스턴스 변수를 작성 합니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class RandomWordsState extends State<RandomWords> {
  final _suggestions = <WordPair>[];
  final _biggerFont = const TextStyle(fontSize: 18.0);
  
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();
    return Text(wordPair.asPascalCase);
  }
}
{% endcodeblock %}

Java와는 달리 Dart에는 public, protected, private 키워드가 없습니다. 변수 식별자가 underscore(_)로 시작하면 해당 변수는 해당 라이브러리 내부에서만 사용할 수 있습니다. Java의 private와 유사합니다.

RandomWordsState 클래스에 _buildSuggestions() 함수를 추가합니다. 이것은 단어를 표시하는 ListView를 작성합니다.

ListView클래스는 익명함수로 지정된 Factory Builder 및 Callback 함수인 builder와 itemBuilder를 제공합니다.
ItemBuilder는 2개의 인자를 받는데요, BuildContext와 반복자 i가 전달됩니다. 반복자 i는 0에서 시작해서 해당 함수가 호출될때마다 계속 증가 됩니다. 해당 함수는 스크롤 할때마다 호출이 됩니다.
그리고 단어를 표시해주는 ListTile Widget을 홀수로 구역을 분리해주는 선인 Divider는 짝수별로 생성합니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
Widget _buildSuggestions() {
    return ListView.builder(
      itemBuilder: (context, i) {
        // 반복자 i가 짝수일때 divider를 반환한다.
        if(i.isOdd) return Divider();
        
        // 이 표현식은 i를 2로 나누고 나는 목의 값을 반환합니다.
        final index = i ~/ 2; 
        // List에 표시되는 단어의 수가 _suggestion수보다 클때 다시 _suggestion에 10개의 단어를 추가합니다.
        if(index >= _suggestions.length) {  
          _suggestions.addAll(generateWordPairs().take(10));
        }
        return _buildRow(_suggestions[index]);
      }
    );
  }
{% endcodeblock %}

_buildSuggestions함수는 단어당 한번씩 _buildRow()함수를 호출 합니다. _buildRow함수를 만들어 봅니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
Widget _buildRow(WordPair pair) {
  return ListTile(
    title: Text(
      pair.asPascalCase,
      style: _biggerFont,
    ),
  );
}
{% endcodeblock %}

RandomWordsState 클래스에서 단어를 생성하는 부분을_buildSuggestions()를 사용해서 ListView Widget을 반환하도록 수정합니다. Scaffold도 Widget이기 때문에 이전 MyApp에서 사용했던 Scaffold를 RandomWordState에서도 사용할 수 있습니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title : Text('infinite Scroll Word'),
    ),
    body : _buildSuggestions()
  );
}
{% endcodeblock %}

그리고 MyApp의 build부분의 MaterialApp의 home속성 부분을 stateful widget RandomWords으로 수정해줍니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
  
    final wordPair = WordPair.random();
    
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: RandomWords()
    );
  }
}
{% endcodeblock %}

여기까지 이상없이 작성하셨으면 무한 스크롤되는 앱이 동작을 하게 됩니다.

지금 까지 실행한 앱은 디버그 모드로 작성이 되어서 여러 큰 리소스를 사용하여 빠르게 개발을 도와줍니다. 만약에 앱을 출시하거나 성능을 분석하기 위해서라면 Flutter’s profile or release builds를 사용해야 합니다. 

자세한 정보는 여기(https://flutter.dev/docs/testing/build-modes) 에서 참고 하시기 바래요.

다음 장에서는 해당 앱에 기능을 더 붙이도록 하겠습니다.

읽어주셔서 감사합니다. 좋은 하루 보내세요.