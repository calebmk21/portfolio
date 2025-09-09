---
title: This is a Corporate Worker Simulator Code Snippet
---

## Preface
This is a sample of the Game Manager scripting I did for this project. The full GitHub page can be found here: https://github.com/calebmk21/GMTK-25/tree/main
In addition to the Game Manager, I also implemented the dialogue via YarnSpinner. Due to the nature of the game jam, I did not fully implement everything, as we prioritized finishing a build for the game. I wrote several custom methods to interface between YarnSpinner and the rest of the Unity project.

### Snippet
```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.UI;
using TMPro;
using Unity.VisualScripting;
using UnityEngine.SceneManagement;
using Yarn;
using Yarn.Unity;
public class GameManager : MonoBehaviour
{
    // Allows this to be referenced elsewhere
    public static GameManager Instance;
    public DialogueRunner dialogueRunner;
    // Length in seconds of the work day
    /*
     QUESTION: Should this be the timer to make a decision or the total timed workday
     i.e. you have workdayLength seconds to complete your task OR the minigame and you can
     flip between tasks 
     ALTERNATIVELY we can have this be the timer to make a decision and you get locked into said
     decision and must finish it (to success or fail depending on performance), then it moves to the next segment
     */
    //[SerializeField] public float workdayLength = 120f;
    
    // Number of workdays playable
    [SerializeField] public int maxWorkdays = 8;

    // Day number
    [SerializeField] public int dayNumber;
    
    // Action counter
    [SerializeField] int actionsRemaining;

    [SerializeField] public GameObject minigamePanel; 
    
    // Route Points
    public int greedPoints;
    public int slothPoints;
    public int pridePoints;
    public int wrathPoints;
    public int gluttonyPoints;
    public int envyPoints;
    public int lustPoints;
    
    
    // Game State variables
    public GameState State;
    public RouteState Route;
    public MinigameState Minigame;

    public string routeString;
    public string stateString;

    // Events
    public static event Action<GameState> OnGameStateChanged;
    public static event Action<RouteState> OnRouteStateChanged;
    public static event Action<MinigameState> OnMinigameSelect;
    public static event Action OnDay1;
    public static event Action OnDay2;
    public static event Action OnSnooze;
    
    void Awake()
    {
        Instance = this;
    }

    // interfacing with yarn variables
    private InMemoryVariableStorage variableStorage;
    
    void Start()
    {
        UpdateGameState(GameState.Tutorial);
        variableStorage = FindFirstObjectByType<InMemoryVariableStorage>();
        //Debug.Log("Actions Remaining: " + actionsRemaining);
    }

    // Handles logic for which route you're changing to
    public void RouteSelector()
    {
        int[] routeArray = new int[7]
            { greedPoints, slothPoints, pridePoints, wrathPoints, gluttonyPoints, envyPoints, lustPoints };
        
        // Quick Helper functions
        int MaxValue(int[] arr)
        {
            int max = 0;
            for (int i = 0; i < arr.Length; i++)
            {
                if (arr[i] > max)
                {
                    max = arr[i];
                }
            }

            return max;
        }
        bool MaxDuplicate(int[] arr, int max)
        {
            bool dupe = false;
            int maxCounts = 0;
            
            
            for (int i = 0; i < arr.Length; i++)
            {
                if (arr[i] == max)
                {
                    maxCounts++;
                }
            }

            if (maxCounts > 1)
            {
                dupe = true;
            }
            
            return dupe;
        }
        int RouteToInt(RouteState route)
        {
            switch (route)
            {
                case RouteState.Greed:
                    return 0;
                case RouteState.Sloth:
                    return 1;
                case RouteState.Pride:
                    return 2;
                case RouteState.Wrath:
                    return 3;
                case RouteState.Gluttony:
                    return 4;
                case RouteState.Envy:
                    return 5;
                case RouteState.Lust:
                    return 6;
                case RouteState.Indecisive:
                    return 7;
                default:
                    throw new ArgumentOutOfRangeException(nameof(route), route, null);
            }
        }
        RouteState IntToRoute(int i)
        {
            switch (i)
            {
                case 7:
                    return RouteState.Indecisive;
                case 0:
                    return RouteState.Greed;
                case 1:
                    return RouteState.Sloth;
                case 2:
                    return RouteState.Pride;
                case 3:
                    return RouteState.Wrath;
                case 4:
                    return RouteState.Gluttony;
                case 5:
                    return RouteState.Envy;
                case 6:
                    return RouteState.Lust;
                default:
                    throw new ArgumentOutOfRangeException(nameof(i), i, null);
            }
        }
        int MaxElementIndex(int[] arr)
        {
            int ind = 0;
            for (int i = 0; i < arr.Length; i++)
            {
                if (arr[i] > arr[ind])
                {
                    ind = i;
                }
            }
            return ind;
        }

        //Debug.Log("Route Array: " + routeArray[0] + routeArray[1] + routeArray[2] + routeArray[3] + routeArray[4] + routeArray[5] + routeArray[6]);
        
        // Gets max value from the array
        int max = MaxValue(routeArray);
        bool dupe = MaxDuplicate(routeArray, max);
        int routeInt = RouteToInt(Route);

        //.Log("Max Value in Array: " + max);
        //Debug.Log("Is the Max Value Duplicate? " + dupe);
        //Debug.Log("Route Int: " + routeInt);
        
        RouteState newRoute = Route;

        // if currently in indecision route, routeInt is out of scope for routeArray;
        // this case handles that first
        if (routeInt == 7)
        {
            //Debug.Log("Entered 1");
            if (dupe)
            {
                //Debug.Log("Entered 1.1");
                newRoute = RouteState.Indecisive;
            }
            else
            {
                //Debug.Log("Entered 1.2");
                int ind = MaxElementIndex(routeArray);
                //Debug.Log("Index: " + ind);
                newRoute = IntToRoute(ind);
            }
        }
        
        // Ties in points go to the current route
        else if (dupe && routeArray[routeInt] == max)
        {
            //Debug.Log("Entered 2");
            newRoute = Route;
        }
        
        // Unless you tied two new routes at the same time
        else if (dupe && routeArray[routeInt] != max)
        {
            //Debug.Log("Entered 3");
            newRoute = RouteState.Indecisive;
        }
        else
        {
            //Debug.Log("Entered 4");
            int ind = MaxElementIndex(routeArray);
            newRoute = IntToRoute(ind);
        }
        
        RouteChange(newRoute);
        
    }
    
    // Handles what happens when you change routes

    public void RouteChange(RouteState newRoute)
    {
        Route = newRoute;
        switch (newRoute)
        {
            case RouteState.Indecisive:
                // Defaults to Greed on Day 1
                if (dayNumber == 1)
                {
                    RouteChange(RouteState.Greed);
                }
                break;
            case RouteState.Greed:
                break;
            case RouteState.Sloth:
                break;
            case RouteState.Pride:
                break;
            case RouteState.Wrath:
                break;
            case RouteState.Gluttony:
                break;
            case RouteState.Envy:
                break;
            case RouteState.Lust:
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(newRoute), newRoute, null);
        }
        
        OnRouteStateChanged?.Invoke(newRoute);
    }
    
    public void UpdateGameState(GameState newState)
    {
        State = newState;
        switch (newState)
        {
            case GameState.Morning:
                // Actions resets only when morning is called
                HandleMorning();
                break;
            case GameState.Workday:
                HandleWorkday();
                break;
            case GameState.Minigame:
                // Selecting a minigame decrements the action counter
                HandleMinigame();
                break;
            case GameState.Evening:
                HandleEvening();
                break;
            case GameState.Ending:
                HandleEnding(Route);
                break;
            case GameState.Tutorial:
                HandleTutorial();
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(newState), newState, null);
        }
        
        OnGameStateChanged?.Invoke(newState);
    }

    public void SetGreedEndingStatusWithoutTheBS()
    {
        State = GameState.Ending;
        Route = RouteState.Greed;
    }
    
    // Game State Functions

    public void HandleMorning()
    {
        // Increments day counter
        ++dayNumber;
        Debug.Log("Day: " + dayNumber);        
        
        string yarnNode = "";

        switch (dayNumber)
        {
            case 2:
                yarnNode = "DayTwo";
                break;
            case 3:
                yarnNode = "DayThree";
                break;
            case 4:
                yarnNode = "DayFour";
                break;
            case 5:
                yarnNode = "DayFive";
                break;
            case 6:
                yarnNode = "DaySix";
                break;
            case 7:
                yarnNode = "DaySeven";
                break;
            case 8:
                yarnNode = "DayEight";
                break;
            default:
                yarnNode = "DayOneMorning";
                break;
        }
        
        
        // Unique events for day 1 and day 2 (cut due to scope)
        // Day 1 NO LONGER only gets one action
        if (dayNumber == 1)
        {
            OnDay1?.Invoke();
            actionsRemaining = 3;
        }
        // else if (dayNumber == 2)
        // {
        //     OnDay2?.Invoke();
        //     actionsRemaining = 3;
        // }
        
        // Begins ending if you reached the final day
        else if (dayNumber >= maxWorkdays)
        {
            UpdateGameState(GameState.Ending);
        }
        else
        {
            Debug.Log("Yarn Node: "+ yarnNode);
            actionsRemaining = 3;
            dialogueRunner.StartDialogue(yarnNode);
        }
        

    }

    public void HandleWorkday()
    {
        Debug.Log("Number of Actions: " + actionsRemaining);
        Debug.Log("Currently on Day: "+ dayNumber);
        
        if (dayNumber >= maxWorkdays)
        {
            UpdateGameState(GameState.Ending);
        }
        else if (actionsRemaining <= 0)
        {
            // Change to evening later if need be
            UpdateGameState(GameState.Evening);
        }
        else
        {
            minigamePanel.SetActive(true);
        }
        
    }

    public void HandleMinigame()
    {
        actionsRemaining--;
    }

    public void HandleEvening()
    {
        RouteSelector();
        SendToYarn();
        UpdateGameState(GameState.Morning);
    }

    public void SendToYarn()
    {   
        routeString = RouteStateToString(Route);
        stateString = GameStateToString(State);
        variableStorage.SetValue("$greed_points", greedPoints);
        variableStorage.SetValue("$sloth_points", slothPoints);
        variableStorage.SetValue("$pride_points", pridePoints);
        variableStorage.SetValue("$envy_points", envyPoints);
        variableStorage.SetValue("$route", routeString);
        variableStorage.SetValue("$state", stateString);
        variableStorage.SetValue("$day", dayNumber);
    }
    

    public void HandleEnding(RouteState finalRoute)
    {
        switch (finalRoute)
        {
            case RouteState.Indecisive:
                SceneManager.LoadScene("IndecisionEnding");
                break;
            case RouteState.Greed:
                SceneManager.LoadScene("GreedEnding");
                break;
            case RouteState.Sloth:
                SceneManager.LoadScene("SlothEnding");
                break;
            case RouteState.Pride:
                SceneManager.LoadScene("PrideEnding");
                break;
            case RouteState.Wrath:
                break;
            case RouteState.Gluttony:
                break;
            case RouteState.Envy:
                SceneManager.LoadScene("EnvyEnding");
                break;
            case RouteState.Lust:
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(finalRoute), finalRoute, null);
        }
    }

    // This should only be called if the player wishes to replay the tutorial
    public void HandleTutorial()
    {
        
    }

    public string GameStateToString(GameState state)
    {
        string s = "";

        switch (state)
        {
            case GameState.Morning:
                s = "Morning";
                break;
            case GameState.Workday:
                s = "Workday";
                break;
            case GameState.Minigame:
                s = "Minigame";
                break;
            case GameState.Evening:
                s = "Evening";
                break;
            case GameState.Ending:
                s = "Ending";
                break;
            case GameState.Tutorial:
                s = "Tutorial";
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(state), state, null);
        }
        return s;
    }
    public string RouteStateToString(RouteState route)
    {
        string s = "";
        switch (route)
        {
            case RouteState.Indecisive:
                s = "Indecisive";
                break;
            case RouteState.Greed:
                s = "Greed";
                break;
            case RouteState.Sloth:
                s = "Sloth";
                break;
            case RouteState.Pride:
                s = "Pride";
                break;
            case RouteState.Wrath:
                s = "Wrath";
                break;
            case RouteState.Gluttony:
                s = "Gluttony";
                break;
            case RouteState.Envy:
                s = "Envy";
                break;
            case RouteState.Lust:
                s = "Lust";
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(route), route, null);
        }

        return s;
    }
    
    
    public void MinigameSelection(MinigameState minigame)
    {
        Minigame = minigame;
        switch (minigame)
        {
            case MinigameState.Sleep:
                slothPoints++;
                Debug.Log("Sloth Points: " + slothPoints);
                break;
            case MinigameState.Spreadsheet:
                // greedPoints++;
                Debug.Log("Greed Points: " + greedPoints);
                break;
            case MinigameState.Match:
                
                // Point gain varies based on which route you are on; defaults to greed
                // Stronger gains if you're already on the non-greed route
                if (Route == RouteState.Envy)
                {
                    envyPoints += 2;
                }
                else if (Route == RouteState.Pride)
                {
                    pridePoints += 2;
                }
                else
                {
                    pridePoints += 1;
                    envyPoints += 1;
                }
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(minigame), minigame, null);
        }
        
        OnMinigameSelect?.Invoke(minigame);
    }
    
    public enum GameState
    {
        Morning,
        Workday,
        Minigame,
        Evening,
        Ending,
        Tutorial
    }

    public enum MinigameState
    {
        Sleep,
        Spreadsheet,
        Match
    }

    public enum RouteState
    {
        Indecisive,
        Greed,
        Sloth,
        Pride,
        Wrath,
        Gluttony,
        Envy,
        Lust
    }


    public int greedNumber = 0;
    public string greedString = "";
    public void AdvanceGreedDialogue()
    {
        greedNumber++;
        greedString = "Greed" + greedNumber.ToString();
        Debug.Log("Greed String: " + greedString);
        dialogueRunner.StartDialogue(greedString);
    }
    
}
```

### Yarn Snippet
```
title: TutorialIntro
position: -358,-567
---
<<declare $greed_points = 0>>
<<declare $sloth_points = 0>>
<<declare $pride_points = 0>>
<<declare $envy_points = 0>>
<<declare $route = "Greed">>
<<declare $state = "Tutorial">>
<<declare $day = 0>>
<<declare $first_minigame = "Spreadsheet">>

Greed: Once upon a time, there was a loyal, hardworking employee named… #line:Greed_Tutorial_1
Greed: … #line:blank
Greed: Well, that’s truly embarrassing. It appears that I didn’t receive ANY information about your personal details. I’m deeply sorry, but would you mind telling me your name? #line:Greed_Tutorial_2
Greed: I see. Duly noted for possible usage later! Before I say anything else, congratulations on being hired at Company Co! Truly, this is a marvelous accomplishment for someone straight out of university with almost nothing to their name. #line:Greed_Tutorial_3
Greed: Now, it’s time for you to show the world just what exactly you can do! Get out of bed, and let’s get to work! #line:Greed_Tutorial_4
<<jump TutorialBed>>
===

title: TutorialBed
position: -357,-355
---
-> You: Snooze the alarm.
Greed: Hahaha! Oh, how amusing! It seems you misheard me when I told you to NOT press the snooze button! Common mistake, really. Let’s try that again. #line:Greed_OoBTutorial_1
Greed: Now, get up! #line:Greed_OoBTutorial_2.1
-> You: Snooze the alarm again. 
Sloth: *yawns* five more minutes please. that's all I ask. #line:Sloth_Tutorial_1
Greed: Oh God. Ignore him, please. Believe me, he’s the last thing you’d want as an influence on your work ethic. #line:Greed_OoBTutorial_3
Greed: I have to say, this is NOT a good look for your first day; you’re going to be late! I’m getting you out of bed, whether you like it or not. You have very important tasks you have to learn today! #line:Greed_OoBTutorial4
<<open_workstation>>
===
title: Orientation
position: -362,-129
---
Tutorial: Welcome to your new job at Company Co. 
Tutorial: Here you can perform one of three different actions.
Tutorial: You can work on Spreadsheets, which is part of your job.
Tutorial: Gain as many points as possible without trapping yourself inside the spreadsheets.
Tutorial: You can send out Emails, which is the other part of your job.
Tutorial: A memory matching game where you can either sabotage your coworkers (green) or build your own business (blue).
Tutorial: Or, yknow. Do your job (yellow).
Tutorial: Lastly, you can listen to the eepy voice in your head and Nap.
Tutorial: Click the red Z's to stay asleep longer. 
Tutorial: It's up to you to decide how you want to spend your first 7 days here.
Tutorial: Depending on your actions, you may get one of five different endings. 
Tutorial: This concludes your orientation.
Tutorial: From now on, you'll only hear from the voices in your head.
===


title: PostOrientation
position: -362,89
---
Greed: See, that wasn't too bad. You only have to do this every day for the rest of your life! You'll be rich in no time. #line:Greed_Orientation_1
Sloth: can we sleep in tomorrow? #line:Sloth_Tutorial_2
Greed: Oh God you're back. Why are you back? Move along now. Tah-Tah! #line:Greed_Orientation_2
Greed: ...
Greed: He's gone? Good. Okay, so you're currently on a probationary period. In 7 day's time, you'll have your first performance review. Be a good little employee until then, okay? We can't have any mishaps or ulterior motives jeopardizing this WONDERFUL opportunity you have now, can we? #line:Greed_Orientation_3
<<to_game>>
===
```

### Custom Yarn Commands
```
using System;
using UnityEngine;
using UnityEngine.SceneManagement;
using Yarn.Unity;

public class YarnHandler : MonoBehaviour
{

    public DialogueRunner dialogueRunner;

    public GameObject workstation;
    
    void Awake()
    {
        dialogueRunner.AddCommandHandler(
            "var_to_yarn",
            SendVariablesToYarn);
        dialogueRunner.AddCommandHandler(
            "to_workday",
            ToWorkday
            );
        dialogueRunner.AddCommandHandler(
            "to_morning",
            ToMorning
        );
        dialogueRunner.AddCommandHandler(
            "to_evening",
            ToEvening
            );
        dialogueRunner.AddCommandHandler(
            "to_game",
            ToMainGame);
        dialogueRunner.AddCommandHandler(
            "open_workstation",
            OpenWorkstation);
        dialogueRunner.AddCommandHandler(
            "close_workstation",
            CloseWorkstation);
        dialogueRunner.AddCommandHandler(
            "greed_ending",
            SetGreedEnding);
        
    }
    
    // Yarn Commands

    public void SetGreedEnding()
    {
        GameManager.Instance.SetGreedEndingStatusWithoutTheBS();
    }
    
    public void CloseWorkstation()
    {
        workstation.SetActive(false);
        Debug.Log("Closed Workstation");
    }
    public void OpenWorkstation()
    {
        workstation.SetActive(true);
    }

    public void ToMainGame()
    {
        // may change if necessary
        SceneManager.LoadScene("GameManagerTest");
    }
    
    public void SendVariablesToYarn()
    {
        GameManager.Instance.SendToYarn();
    }

    public void ToWorkday()
    {
        GameManager.Instance.UpdateGameState(GameManager.GameState.Workday);
    }

    public void ToMorning()
    {
        GameManager.Instance.UpdateGameState(GameManager.GameState.Morning);
    }

    public void ToEvening()
    {
        GameManager.Instance.UpdateGameState(GameManager.GameState.Evening);
    }

    public void PlayNewNode(string nodeName)
    {
        dialogueRunner.StartDialogue(nodeName);
    }
    
    
    
}
```