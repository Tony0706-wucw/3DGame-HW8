### UI 效果制作
•进入 NGUI 官方网站，使用 UGUI 实现以下效果
**• Quest Log 公告牌**
•以上例子需要使用 Unity web player， 仅支持以下操作系统与浏览器，参见官方下载
• Windows 版 IE11
• Mac OS X 10.7 Safari
• 出现界面需要等待较长时间，打开页面让它慢慢加载  

---

在Scene场景中添加ScrollView游戏对象，设置属性：取消了滚动视图的水平滚动，设置竖直滚动条永远可见。

![image-20221211132633934](C:\Users\tony0706\AppData\Roaming\Typora\typora-user-images\image-20221211132633934.png)

对Content列表内容进行设置：添加 `Verticle Layout Group` 和 `Content Size Fitter` 组件，前者用于构建垂直列表视图，后者用于实现列表高度的自适应。

![image-20221211131604282](C:\Users\tony0706\AppData\Roaming\Typora\typora-user-images\image-20221211131604282.png)

消息标题按钮的预制：

![image-20221211131630614](C:\Users\tony0706\AppData\Roaming\Typora\typora-user-images\image-20221211131630614.png)

消息主体内容的预制：

![image-20221211131639553](C:\Users\tony0706\AppData\Roaming\Typora\typora-user-images\image-20221211131639553.png)

—

Button 消息标题添加的 `ButtonHandler` 脚本：

```c#
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

public class ButtonHandler : MonoBehaviour
{
    // 消息主体。
    public Text text;
    // 动画帧率。
    public int frame = 5;
    // 消息主体 Text 的高度。
    private float textHeight;
    // 消息主体 Text 的宽度。
    private float textWidth;

    void Start()
    {
        // 绑定响应点击事件。
        GetComponent<Button>().onClick.AddListener(OnClick);
        // 记录消息主体 Text 的高度。
        textHeight = text.rectTransform.sizeDelta.y;
        // 记录消息主体 Text 的宽度。
        textWidth = text.rectTransform.sizeDelta.x;
    }

    // 响应点击事件。
    void OnClick()
    {
        // 当消息主体已被展开，播放关闭动画。
        if (text.gameObject.activeSelf)
        {
            StartCoroutine("CloseText");
        }
        else // 当消息主体未被展开，播放展开动画。
        {
            StartCoroutine("OpenText");
        }
    }

    // 播放关闭消息主体动画。
    private IEnumerator CloseText()
    {
        // 设置旋转角度的初始值和旋转速度。
        float angleX = 0;
        float angleSpeed = 90f / frame;
        // 设置 Text 初始高度和高度缩放速度。
        float height = textHeight;
        float heightSpeed = textHeight / frame;

        // 执行动画。
        for (int i = 0; i < frame; ++i)
        {
            // 更新旋转角度。
            angleX -= angleSpeed;
            // 更新 Text 高度。
            height -= heightSpeed;
            // 应用新的旋转角度和高度。
            text.transform.rotation = Quaternion.Euler(angleX, 0, 0);
            text.rectTransform.sizeDelta = new Vector2(textWidth, height);
            // 结束动画。
            if (i == frame - 1)
            {
                text.gameObject.SetActive(false);
            }
            yield return null;
        }
    }

    // 播放打开消息主体动画。
    private IEnumerator OpenText()
    {
        // 设置旋转的初始值和旋转速度。
        float angleX = -90f;
        float angleSpeed = 90f / frame;
        // 设置 Text 初始高度和高度缩放速度。
        float height = 0;
        float heightSpeed = textHeight / frame;

        // 执行动画。
        for (int i = 0; i < frame; ++i)
        {
            // 更新旋转角度。
            angleX += angleSpeed;
            // 更新 Text 高度。
            height += heightSpeed;
            // 应用新的旋转角度和高度。
            text.transform.rotation = Quaternion.Euler(angleX, 0, 0);
            text.rectTransform.sizeDelta = new Vector2(textWidth, height);
            // 结束动画。
            if (i == 0)
            {
                text.gameObject.SetActive(true);
            }
            yield return null;
        }
    }
}
```

`Content` 添加的 `Content.cs` 脚本：

```c#
using UnityEngine;
using UnityEngine.UI;

public class Content : MonoBehaviour
{
    // num 指消息总数。
    public int num = 10;
    // 列表布局。
    private RectTransform content;

    void Start()
    {
        content = GetComponent<RectTransform>();
        // 加载列表资源。
        LoadResources();
    }

    // 加载列表资源。
    void LoadResources()
    {
        for (int i = 0; i < num; ++i)
        {
            // 设置消息标题。
            Button title = Instantiate(Resources.Load<Button>("Prefabs/Button"));
            title.name = "标题 " + (i + 1);
            title.GetComponentInChildren<Text>().text = "Message Title " + (i + 1);
            // 添加 Button 至消息列表。
            title.transform.SetParent(transform, false);

            // 设置消息主体。
            Text body = Instantiate(Resources.Load<Text>("Prefabs/Text"));
            body.name = "主体 " + (i + 1);
            body.text = "消息 " + (i + 1) + ".";
            // 添加 Text 至消息列表。
            body.transform.SetParent(transform, false);
            body.gameObject.SetActive(false);

            // 设置 Button 响应点击事件。
            ButtonHandler handler = title.gameObject.AddComponent<ButtonHandler>();
            handler.text = body;
        }
    }
}
```

最终呈现效果：

![image-20221211131113759](C:\Users\tony0706\AppData\Roaming\Typora\typora-user-images\image-20221211131113759.png)