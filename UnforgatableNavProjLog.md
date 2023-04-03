# 场景切换
## 创建scene，设置camera
<font size=4>创建两个scene，因为每个scene自带camera，如果需要第一个呈现的scene1背景不呈现第二个scene，那么scene1的camera，其Depth变量数值需设置大于scene2的camera -> Depth数值。  
    如图：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318151527.png" alt="camera->Depth" width="300"/> <img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318151856.png" width="300"/>
    <font>
## Build settings  
<font size=4>打开unity -> File ->Build settings。  
    <img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318152132.png" alt="Unity Build settings" width="300"/>  
    按照场景切换的顺序设置Scenes in Build，如果没有则Add OpenScenes，然后build。  
    <img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318152511.png" alt="Build" width=400/>
    <font>
## 代码部分  
<font size=4>UnityEngine.Scenemanagement中有相关函数。</font>  

# 收集被试信息
## InputField
<font size=4>unity -> 右键 -> UI -> Input Field，然后在Inspector -> On..中导入挂载有脚本的GameObject及脚本的方法。如图：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318193536.png" width=350/> 
    <font>
## 将信息写入csv文件  
<font size=4>UI -> Button，将Button的点击（On Click()）与写入csv文件的方法相关联：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230330163632.png"/><img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230330163700.png"/>
    <font>
## 代码部分  
<font size=4>见相关脚本。  
    **note**：c#的StreamWriter有一个自己的缓冲区，缓冲区满之后才会写入文件中，因此最好每次写入使用using限定scope，退出该scope时直接flush缓冲区，把数据写入文件。举例如下：  
    <font>
```c#
    using(StreamWriter writer = new StreamWriter(sub_info))
        {
            string headerString = string.Join(",", headers);
            writer.WriteLine(headerString);
            print(headerString);
            //create and write rows
            string[] rows = { group_ID.ToString(), ID.ToString(), Name, Age.ToString(), gender.ToString() };
            string personal_info = string.Join(",", rows);
            writer.WriteLine(personal_info);
        }
```

# 收集物体（学习物体位置阶段）
## GameEvents & Listener & Trigger
<font size=4>设置一个Action`public event Action<int> OnFindTheObject;`，可以被subscribe。通过触发`GameEvents.current.FindTheObject(id);`可以调用其中的对应Action，然后Action会检查所有订阅了该Action的Listener。  <font>
    
```c#
    if (OnFindTheObject != null)
        {
            OnFindTheObject(id);//check every Listener subscribing to the Action set
        }
```
<font size=4>如果在之前给Listener设置了ID，则可以使用ID决定唤醒哪一个Listener并执行对应活动。</font>
## 物体的自动移动  
<font size=4>见ObjectMover.cs。善用vector3函数和transform.position属性。
    <font>
## 物体头上呈现名字+物体朝向
<font size=4>3D Object -> text + `transform.LookAt()`。注意，面朝摄像机时，朝向是上下左右翻转的（暂时不清楚原因），需要特别设置，详见代码。
    <font>  
## 开始下一个trial（收集下一个物体）  
<font size=4>使用一个bool值作为trial trigger，每次收集物体成功则改变bool值，进入下一个试次（同时reset bool值及player状态）。  
    详见ObjectListener.cs和ExperimentProcessing.cs
    <font>  
## 每个试次的指导语  
### 结构
<font size=4>创建canvas用于显示指导语，添加UI->text显示指导语，UI->button继续下一个试次：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230329190303.png" width=300/>
    <font>  
        
### 设置空格键激活按钮
<font size=4>新建脚本ContinueButton.cs将空格键与ContinueButton相连（按空格便可激活按钮）。将ContinueButton.cs附上GuidanceCanvas，将ContinueButton游戏物体赋给脚本中的Continue游戏物体，即可获得对应Button组件`Continue.GetComponent<Button>()`以激活Button：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230329191034.png"/>  
    具体代码见ContinueButton.cs。  
    <font>  

### TrialManager.cs控制Trial
<font size=4>在TrialManager.cs中，方法`GuidancePause()`修改布尔值`isPaused = true`，以停止键盘输入，呈现每个Trial的指导语，`TrialStart()`撤掉指导语，开始下一个试次。在`LearningPhase()`的每次循环开始时调用`GuidancePause()`，按空格激活Button，Button调用`TrialStart()`，实现每个试次前呈现指导语。  
    note：Continue Button调用`TrialStart()`的实现方法与前文收集被试信息的Start Button实现方法一致，如图：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230330162844.png" width=350/>
    <font>  
# 实验预设  
## 预设读取
<font size=4>预设所在的CSV文件格式如图：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230327153112.png" width=350/>  
    首行为headers，存储要读入的参数名称。GameSetter.cs会自动读取headers并分割存储为string数组，同时也可以自动计算每个header下的行数（也就是有多少个值）并进行读取，不过可惜地是，由于可能存在的对增删改preset.csv的header的需求，header没法固定，所以每次修改都需要去GameSetter.cs中修改对应的header名称：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230327155339.png" width=600/>如图所示，修改目标位置坐标header时需要修改对应函数中的对应header。
    <font>