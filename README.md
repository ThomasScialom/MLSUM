# MLSUM

The original dataset as used in the paper is available on HuggingFace datasets (https://github.com/huggingface/datasets/tree/master/datasets/mlsum)

Usage of dataset is restricted to non-commercial research purposes only.
Copyright belongs to the original copyright holders.

It is also available [here](https://drive.google.com/file/d/1Z4oswHQ8yYPHqxendaC3jAzAfYEP9CbJ/view?usp=sharing).

## Outputs used in the paper 

We make available the [outputs](https://drive.google.com/file/d/1YFlBhEO-yLv28xtAEX28p1JxWfjBKNzn/view?usp=sharing) from our models so anyone can fairly compare. 
Note that if you obtain a different ROUGE, it might be due to the library: 
- for BERT-Gen, we used the [UNILM Rouge](https://github.com/microsoft/unilm/blob/master/unilm-v1/src/cnndm/eval.py)
- for the other moels we used the [ROUGE](https://pypi.org/project/rouge/) pypi library, version 0.3.1

## Instructions and code to rebuild the dataset from the archived web pages:

#### Setup the environment  
```shell
cd MLSUM
conda create --name mlsum
conda activate mlsum
conda install pip
pip install requirements -r
 ```

#### Download the URLs 

The list of URLs is available here:

```shell
https://drive.google.com/file/d/1qViYZwl82yyyTE5mKryhVWbFao4Ck-7g/view?usp=sharing
```

In the main folder MLSUM, create a folder data and unzip the URL folder there. Create also an empty folder processed, in which the data will be stored. 

```shell
mkdir data
cd data
unzip urls.zip
mkdir processed
 ```
    
#### Scrap all the MLSUM data on web.archive

Not that it is possible that some URLs fail to be processed for various reasons. All those failed URLs are listed in the 'data/processed/*.errors.txt' files. 

```shell
python run_all.py
```

## Reproducing the results

### Training BERT-gen 

We used the [UniLM code](https://github.com/microsoft/unilm/tree/master/unilm-v1#abstractive-summarization---cnn--daily-mail) for abstractive summarization with the default parameters except:
- num_train_epochs set to 5 (instead of 30)
- model_recover_path is simply the [multilingal BERT checkpoint](https://huggingface.co/bert-base-multilingual-uncased/tree/main) (instead of unilmv1-large-cased.bin)


### Russian Score:

It seems that for Russian, the results are very different given the implementation of ROUGE metric.
To reproduce the one used in the paper, install the following ROUGE package:

```shell
pip install rouge==0.3.1
```  

Then, the bellow script should give you results corresponding results: 

```python
from rouge import Rouge

def get_rouge(hypothesis, references):
    rouge = Rouge()
    preprocess_exs = lambda exs : [ex.strip().lower() for ex in exs]
    rouge_scores =  rouge.get_scores(preprocess_exs(hypothesis), preprocess_exs(references), avg=True)
    return {k: v['f'] for k, v in rouge_scores.items()}
    
refs = ['Старший преподаватель института коммунального хозяйства и строительства был задержан на днях в Москве за растление школьника',
    'Манежная площадь Москвы стала местом последнего в 2009 году убийства',
    'Президент РФ Дмитрий Медведев с семьей проводит новогодние праздники на горнолыжном курорте “Красная Поляна”, а в воскресенье к нему в гости приехал и премьер Владимир Путин']

gens = ['Миллениалы , которые не знают , уходит электричество из розетки или нет , если выключить свет , крайне обрадовались , когда недавно Илон Маск вывел на орбиту первые 60 спутников для интернет-сети Starlink . Основной посыл — началось ! Скоро у нас везде будет бесплатный спутниковый Интернет , до которого не дотянутся руки Роскомнадзора .', 
    'Если верить южнокорейскому изданию , ссылающемуся на анонимные источники , спецпредставитель Ким Хёк Чхоль и четверо неназванных сотрудников Министерства иностранных дел КНДР были казнены в марте в Пхеньяне на военном аэродроме Мирим . Напомним , что встреча на высшем уровне между лидерами Соединенных Штатов и Северной Кореи во вьетнамской столлице , на которую Трамп возлагал , судя по всему , немалые надежды , была закончена раньше намеченного срока . Сторонам не удалось ни о чем договориться , и никаких соглашений по ядерному разоружению Пхеньяна подписано не было .',
    'ЦИТАТА ДНЯ Андрей ВОРОБЬЕВ : « Наша ключевая задача — сделать так , чтобы люди , вызвавшие « скорую » , могли точно знать , когда к ним приедет бригада . Такой сервис есть в Европе . Должен быть и у нас » .']

print(get_rouge(gens, refs))
```

*Output:*
```
{'rouge-1': 0.05170222555648688, 'rouge-2': 0.0, 'rouge-l': 0.04330277388057737}
```
