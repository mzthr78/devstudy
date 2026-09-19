#

ロードするシーンをBuild ProfilesのScene Listに追加する

![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/menu_file_buildprofiles.png)
![](https://github.com/mzthr78/docs/blob/master/dev/unity/image/buildprofiles.png)


```
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneChange : MonoBehaviour
{
    public void LoadScene(string sceneName)
    {
        SceneManager.LoadScene(sceneName);
    }
}
```
