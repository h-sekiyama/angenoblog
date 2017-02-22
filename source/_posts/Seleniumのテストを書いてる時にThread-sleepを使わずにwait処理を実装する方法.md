---
title: Seleniumのテストを書いてる時にThread.sleepを使わずにwait処理を実装する方法
date: 2017-02-20 13:27:00
tags: selenium
---
どうも、せっきーです。  
いま業務でseleniumを使ったUI自動テストの実装を進めています。  
  
seleniumは色んな言語で記述できるし、何ならユーザーの動作を記録して自動でテストケースを生成したり、色々できるんですが、その辺の説明は他のブログにお任せします。  
  
今回やりたい事は、タイトルの通りですが、Thread.sleepを使わずに一定時間waitする処理を自前で作る方法についてです。  
また、テストケースを記述する言語はJAVAになります。  
  
seleniumでは、何らかのアクション（ボタンのクリックなど）でDOMが変化し、そのDOMの変化を待ってからその値を検証する様なテストケースの場合、その「◯◯が終わるまで待機」と言う処理を実装する方法はいくつかあるのですが、一般的に多く使われるのは、ExpectedConditionsを使ったwait処理ではないかと思います。  

ここでは詳しくは説明しませんが、例えば  

```
wait.until(ExpectedConditions.titleIs("ページタイトル"));  
```
の様に記述することで、ページタイトルが指定の文字列になるのを待ったり出来ます。  

ExpectedConditionsのメソッドは結構色んな種類があるので、大抵の待機条件は作れます。  

でも、今回どうしてもオリジナルの待機条件を作る必要があったので、ExpectedConditionsクラスを拡張して、「◯◯秒経ったら次の処理を開始する」為のクラスを作成しました。  
  
Thread.sleepを使えば同様の処理は行えるのですが、どうもThread.sleepはスレッドを停止させる事でバグの原因になる事があるらしいのと、あとコードチェックに使っているSonarQubeでバグとして警告されてしまうので、使えなかったのです。  


で、具体的な記述方法ですが、  

**WaitTimerAttributeCondition.java**    

```
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedCondition;

import java.util.Timer;
import java.util.TimerTask;

public class WaitTimerAttributeCondition implements ExpectedCondition<Boolean> {

  private Boolean flag = false;

  protected WaitTimerAttributeCondition(Long milisec) {
    TimerTask task = new TimerTask() {
      public void run() {
        flag = true;
      }
    };

    Timer timer = new Timer();
    timer.schedule(task, milisec);
  }

  public Boolean apply(WebDriver wd) {
    return flag;
  }
}
```

そして、呼び出し側では、

```
wait.until(new WaitTimerAttributeCondition(5000L));
```

この様に使います。  
上記の例だと、5000ミリ秒、すなわち5秒間処理を待機した後、次の処理を開始します。  

ただこの様に、固定の秒数待機して次の処理を行う方法は、ネットワーク速度やマシンスペックなど環境が異なると、場合によっては上手く動作しない可能性もあります。  

ただ、今回seleniumでの自動テスト対象になったサイトが、bootstrapを使っており、何らかのアクションにいちいちアニメーションが入って来て、そのアニメーションの終了を検知する必要がどうしてもあったので、実装してみました。  

  
  
ぜひ参考にして下さい。