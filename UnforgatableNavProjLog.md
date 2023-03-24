# 场景切换
## 创建scene，设置camera
<font size=4>创建两个scene，因为每个scene自带camera，如果需要第一个呈现的scene1背景不呈现第二个scene，那么scene1的camera，其Depth变量数值需设置大于scene2的camera -> Depth数值。  
    如图：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318151527.png" alt="camera->Depth" width="300"/> <img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318151856.png" width="300"/>
    </font>
## Build settings  
<font size=4>打开unity -> File ->Build settings。  
    <img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318152132.png" alt="Unity Build settings" width="300"/>  
    按照场景切换的顺序设置Scenes in Build，如果没有则Add OpenScenes，然后build。  
    <img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318152511.png" alt="Build" width=400/>
    </font>
## 代码部分  
<font size=4>UnityEngine.Scenmanagement中有相关函数。</font>  

# 收集被试信息
## InputField
<font size=4>unity -> 右键 -> UI -> Input Field，然后在Inspector -> On..中导入挂载有脚本的GameObject及脚本的方法。如图：<img src="https://raw.githubusercontent.com/PsyLinkist/LearningBlogPics/master/20230318193536.png"/>
    </font>
## 代码部分  
<font size=4>见相关脚本。  
    **note**：c#的StreamWriter有一个自己的缓冲区，缓冲区满之后才会写入文件中，因此最好每次写入使用using限定scope，退出该scope时直接flush缓冲区，把数据写入文件。举例如下：  
    </font>
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
# 收集物体（学习物体位置）
## GameEvents & Listener & Trigger
<font size=4>设置一个Action`public event Action<int> OnFindTheObject;`，可以被subscribe。通过Trigger`GameEvents.current.FindTheObject(id);`可以调用其中的对应Action，然后Action会检查所有订阅了该Action的Listener。</font>
    
```c#
    if (OnFindTheObject != null)
        {
            OnFindTheObject(id);//check every Listener subscribing to the Action set
        }
```
<font size=4>如果在之前给Listener分发了ID，则可以使用ID决定唤醒哪一个Listener并执行对应活动。</font>  
## 物体的自动移动  
<font size=4>见ObjectMover.cs。善用vector3函数和transform.position属性。
    </font>
## 物体头上呈现名字+物体朝向
<font size=4>3D Object -> text + `transform.LookAt()`。注意，面朝摄像机时，朝向是上下左右翻转的（暂时不清楚原因），需要特别设置，详见代码。
    </font>  