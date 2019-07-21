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
thumbnailImagePosition: left
thumbnailImage: https://flutter.dev/images/catalog-widget-placeholder.png
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

그리고 반환되는 값을 Scaffold를 사용하고 body를 ListView를 사용해 인자로 divideTitle()을 결과 값을 사용합니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
  void _pushSaved() {
    Navigator.of(context).push(
      MaterialPageRoute<void> (
        builder: (BuildContext context) {
          final Iterable<ListTile> tiles = _saved.map(
              (WordPair pair) {
                return ListTile(
                  title: Text(
                    pair.asPascalCase,
                    style: _biggerFont
                  )
                );
              }
          );
          
          final List<Widget> divided = ListTile.divideTiles(context: context, tiles: tiles).toList();
          
          return Scaffold(
              appBar: AppBar(
                  title: Text('Saved Suggestions')
              ),
              body: ListView(children: divided)
          );
        }
      )
    );
  }
{% endcodeblock %}

단어중 일부를 즐겨찾기에 추가하고 App Bar의 List 아이콘을 탭합니다. 

즐겨찾기한 단어들이 목록으로 존재하는 페이지가 나타납니다. 네비게이터는 App Bar에 자동으로 뒤로 버튼을 추가합니다. 그래서 Navigator.pop()을 명시적으로 구현할 필요가 없습니다. 

홈 버튼으로 돌아가려면 뒤로 버튼을 누릅니다.

### 테마의 변경

테마를 수정하겠습니다. 테마는 앱의 모양이나 느낌을 꾸며줍니다. 물리적 장치나 에뮬레이터에 의존하는 기본 테마를 사용하거나 브랜딩을 위한 테마를 커스텀 할 수 있습니다.

ThemeData 클래스를 구성하여 테마를 쉽게 변경할 수 있습니다. 이 앱은 기본 테마를 사용하지만 앱의 기본 색상을 흰색으로 변경해보도록 하겠습니다.

{% codeblock main.dart lang:dart http://underscorejs.org/#compact main.dart %}
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
  
    final wordPair = WordPair.random();
    
    return MaterialApp(
      title: 'Welcome to Flutter',
      // 추가된 코드
      theme: ThemeData(
          primaryColor: Colors.white
      ),
      home: RandomWords()
    );
  }
}
{% endcodeblock %}

앱을 새로고침하거나 코드를 저장하면 App Bar의 색이 흰색으로 변경된 것을 확인 하실 수 있습니다.

### 우리는 이런것을 해보았어요
 
1. 자연스럽게 보이는 Flutter 앱을 작성하는 방법.
2. 더 빠른 개발 위한 Hot Reload 사용 방법.
3. Stateful Widget에 대화 형 작업을 추가하는 방법.
4. 두 번째 화면을 만들고 탐색하는 방법.
5. 테마를 사용하여 앱의 모양을 변경하는 방법.

다음은 [Introduction to widget](https://flutter.dev/docs/development/ui/widgets-intro)에 대해서 정리를 하려고 합니다.

여기까지 봐주셔서 감사합니다.

다들 좋은 저녁 보내세요.