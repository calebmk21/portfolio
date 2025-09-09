---
title: Scratching By Code Snippet
---

## Preface
This is one of the first projects I began actually scripting in Unity, so my contributions in the programming department were far more limited than in some of my later projects. I primarily wrote this portion of the menu script and implemented some of the UI into the actual project. 

### Snippet
```    
public void navButton(string sceneName)
    {
        SceneManager.LoadScene(sceneName);
    }
    public void quitGame()
    {
        Application.Quit();
        Debug.Log("Quit the application.");
    }
    public void pause()
    {
        pauseMenu.SetActive(true);
        Time.timeScale = 0f;

        foreach (GameObject spawner in spawners) // disable spawners to avoid dragging
        {
            spawner.SetActive(false);
        }

        settings.SwapMusic(0);
        Debug.Log("pause");
    }
    public void resume()
    {
        pauseMenu.SetActive(false);
        Time.timeScale = 1f;

        foreach (GameObject spawner in spawners) // disable spawners to avoid dragging
        {
            spawner.SetActive(true);
        }

        settings.SwapMusic(1);
    }

    public void openPanel()
    {
        if (panel != null)
        {
            bool isActive = panel.activeSelf;
            panel.SetActive(!isActive);
        }


    }
```