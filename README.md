# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе №2 выполнил:
- Шайхутдинов Рамазан Шамильевич;
- РИ-220948;

| Задание | Выполнение | Баллы |
| --- | --- | --- |
| Задание 1 | # | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выберите одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры. Выберите одну из игровых переменных в игре (ресурсы, внутри игровая валюта, здоровье персонажей и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.

Ход работы:
TOTAL WAR: ROME II - компьютерная игра жанра стратегии, создана компанией Creative Assembly. Эта игра захватывает многообразием, здесь решает не только военная смекалка и стратегические маневры, но и правильное управление экономикой, распределение ресурсов, дипломатия, а также стоит строить свою правящую партию и объединяться с другими для избежания раскола внутри собственной империи. Главное концепцией является захват поселений и стирание фракций с лица земли. В качестве игровой переменной я выбрал настроение народа в городах и поселках. Она является важной переменной в этой игре, так как она влияет на рост города, на лечение войнов в городах где они располагаются, при критичном уровне недовольства, рядом с городом будут образовыватся армии мятежников и нападать и грабить города, что плохо скажется на экономике и стратегии в целом. Условия изменения разные, может увеличиваться за счет нахождения армии в городе, построек, особенностей полководцев, избытка еды и так далее. Но может понижаться такжде за счет построек, недостатка пищи в провинции, за счет деятельности агентов, повышенных налогов, а также культурных различий.
Диапазон значений этой переменной - от -100 до 100.
![Экономическая модель в игре Total War: Rome II](https://github.com/SoMNuS65/DA_in_GameDev_Lab_2/blob/main/workshop2.png)

## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в выбранной игре (в качестве таких переменных может выступать игровая валюта, ресурсы, здоровье и т.д.). Средствами google-sheets визуализируйте данные в google-таблице (постройте график, диаграмму и пр.) для наглядного представления выбранной игровой величины.

- Скрипт на Python генерирует рандомное значение общественного порядка в диапазоне от -100 до 100 включительно:
```python
import gspread
import numpy as np
gc = gspread.service_account(filename='urfudatascienceshayhutdinov-7bf018986d8d.json')
sh = gc.open("UnityWorkshop2")
current_mood_of_people = np.random.randint(-100, 101, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = current_mood_of_people[i-1]
        tempInf = str(tempInf)
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(tempInf))
        print(tempInf)
```


## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.
Скрипт для Unity будет запускать звуковые сигналы, в зависимости от значения общественного порядка в городе:
- от -100 до 0 - badSpeak;
- от 0 до 50 - normalSpeak;
- от 50 и выше - goodSpeak.
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] >= 50 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
  if (dataSet["Mon_" + i.ToString()] >= 0 & dataSet["Mon_" + i.ToString()] < 50 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= -100 & dataSet["Mon_" + i.ToString()] < 0 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1z0qvSWXySAzubINPh_jgGU7vhluZhKQi-VedcuDrceU/values/A1:Z1000?key=AIzaSyCIJ8x2wokZwWQe98drFPWfaDX047wMgek");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[1]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```


## Выводы
Ознакомился с заполнением данных в таблицы Google Sheets, с последующей передачей этих данных в Unity для обработки полученных данных и преобразование в звуковые сигналы, в зависимости от входных данных.

## Powered by
Шайхутдинов Р.Ш
