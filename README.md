![WechatIMG3517bf3d58044c60f3609aaeb36c5d79](https://user-images.githubusercontent.com/91502031/143977422-a17282ee-0bae-4bd8-8a8a-2907f30cba22.jpeg)
![WechatIMG4496a8c9a32e436bb4a378bc61504187](https://user-images.githubusercontent.com/91502031/143977449-5913b733-da79-46d6-8556-9b7052e04839.jpeg)

Automatically translate text using Amazon Translate service¶
by David Wei, Zoey Hu, Victoria Shi, Aaron Zhao, Alex Zheng

Globalization is definitely a trend today, with the increasing movement of goods, knowledge and people across borders. Traditional manual operation has been far from meeting the rapidly growing demand for translation, and the demand for machine translation has been unprecedented. machine translation has ushered in a new development opportunity. Machine translation, also known as automatic translation, is the process of using computers to transform one natural source language into another natural target language, generally referring to the translation of sentences and full texts between natural languages.
However, people still wonder whether ML translators are reliable in translation. For example, can we use them when chatting with foreigners on instant messaging platforms? Reading foreign language material, such as poetry and other language-specific texts? How about helping us with our language assignments in an academic setting? Through this project, we were able to dig deeper into Amazon translation services and explore some of the potential vulnerabilities.

Amazon Translate overview¶
Amazon Translate is a neural network machine translation service that provides fast, high-quality, affordable, and customizable language translations. The Amazon Translate service is based on neural networks trained for language translation. This enables you to translate between a source language (the original language of the text being translated) and a target language (the language into which the text is being translated).
When working with Amazon Translate, you will provide source text and get output text:
	•	Source text :The text that you want to translate. You provide the source text in UTF-8 format. 
	•	Output text: The text that Amazon Translate has translated into the target language. Output text is also in UTF-8 format.Depending on the source and target languages, there might be more characters in the output text than in the input text. 
￼

Methods & Data¶
There are several ways we can use Amazon Translate, and the service we choose is “using Amazon Translate to translate large documents”.
We will find English literature, for example, Shakespeare’s literature as source text and translate it to Chinese by both Amazon Translate and Google Translate. Then, we will compare the difference between the output text from these two translators and identify which one is more accurate.
What we should notice is that the data we analyze should be under UTF8 Character encoding and under 5000 bytes in terms of document size. For the document larger than the size limitation, we could use a python program to break the source string into individual sentences to ensure our data is usable.
Architecture Diagram¶
The following shows the various AWS services that were utlizied in this project, including Amazon S3, a bucket storage feature, and SageMaker, the primary tool for training and deploying machine learning model on AWS. Within an EC2 t3 instance, the jupyter notebook was hosted and the command line interface was used to transform data, pass it through Amazon translation, and get a look at simple metrics.
￼

1.Getting Started with Amazon Translate¶
Before using the Amazon Translate service programatically, it will be helpful to understand what it does by waling through examples in the AWS console. To do that, browse through information listed here, then return to this notebook.
In [ ]:
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-1
Default output format [None]: json
In [3]:
import boto3
translate = boto3.client(service_name='translate', region_name='us-east-1', use_ssl=True)

2.By using the Hamret selection as the data set.¶
We selected Hamlet from Shakespeare's plays as the data set and obtained the Chinese data set through human translation. The specific operations are as follows:
In [4]:
#v2.Get Data
orgin_data = []
with open("Hamlet.txt", "r") as f:
    for line in f.readlines():
        line = line.strip('\n')  #去掉列表中每一个元素的换行符
        orgin_data.append(line)

data = []
for i in range(0, len(orgin_data), 2):
    data.append([orgin_data[i], orgin_data[i+1]])
data
Out[4]:
[['SCENE I. A room in the castle.', '第一场 城堡中一室'],
 ['Enter KING CLAUDIUS, QUEEN GERTRUDE, POLONIUS, OPHELIA, ROSENCRANTZ, and GUILDENSTERN',
  '国王、王后、波洛涅斯、奥菲利婭、罗森格兰兹及吉尔登斯吞上。'],
 ['KING CLAUDIUS', '国王'],
 ['And can you, by no drift of circumstance, Get from him why he puts on this confusion, Grating so harshly all his days of quiet With turbulent and dangerous lunacy?',
  '你们不能用迂回婉转的方法，探出他為什么这样神魂颠倒，让紊乱而危险的疯狂困扰他的安静的生活吗？'],
 ['ROSENCRANTZ', '罗森格兰兹'],
 ['He does confess he feels himself distracted; But from what cause he will by no means speak.',
  '他承认他自己有些神经迷惘，可是绝口不肯说為了什么缘故。'],
 ['GUILDENSTERN', '吉尔登斯吞'],
 ['Nor do we find him forward to be sounded, But, with a crafty madness, keeps aloof, When we would bring him on to some confession Of his true state.',
  '他也不肯虚心接受我们的探问；当我们想要引导他吐露他自己的一些真相的时候，他总是用假作痴呆的神气故意回避。'],
 ['QUEEN GERTRUDE', '王后'],
 ['Did he receive you well?', '他对待你们还客气吗？'],
 ['HAMLET', '哈姆莱特'],
 ["To be, or not to be: that is the question: Whether 'tis nobler in the mind to suffer The slings and arrows of outrageous fortune, Or to take arms against a sea of troubles,And by opposing end them? To die: to sleep; No more; and by a sleep to say we end The heart-ache and the thousand natural shocks That flesh is heir to, 'tis a consummation Devoutly to be wish'd. To die, to sleep; To sleep: perchance to dream: ay, there's the rub; For in that sleep of death what dreams may come When we have shuffled off this mortal coil, Must give us pause: there's the respect That makes calamity of so long life; For who would bear the whips and scorns of time, The oppressor's wrong, the proud man's contumely, The pangs of despised love, the law's delay, The insolence of office and the spurns That patient merit of the unworthy takes, When he himself might his quietus make With a bare bodkin? who would fardels bear, To grunt and sweat under a weary life, But that the dread of something after death, The undiscover'd country from whose bourn No traveller returns, puzzles the will And makes us rather bear those ills we have Than fly to others that we know not of? Thus conscience does make cowards of us all; And thus the native hue of resolution Is sicklied o'er with the pale cast of thought, And enterprises of great pith and moment With this regard their currents turn awry, And lose the name of action.--Soft you now! The fair Ophelia! Nymph, in thy orisons Be all my sins remember'd.",
  '生存还是毁灭，这是一个值得考虑的问题；默然忍受命运的暴虐的毒箭，或是挺身反抗人世的无涯的苦难，通过斗争把它们扫清，这两种行為，哪一种更高贵？死了；睡著了；什么都完了；要是在这一种睡眠之中，我们心头的创痛，以及其他无数血肉之躯所不能避免的打击，都可以从此消失，那正是我们求之不得的结局。死了；睡著了；睡著了也许还会做梦；嗯，阻碍就在这儿：因為当我们摆脱了这一具朽腐的皮囊以后，在那死的睡眠里，究竟将要做些什么梦，那不能不使我们踌躇顾虑。人们甘心久困于患难之中，也就是為了这个缘故；谁愿意忍受人世的鞭挞和讥嘲、压迫者的凌辱、傲慢者的冷眼、被轻蔑的爱情的惨痛、法律的迁延、官吏的横暴和费尽辛勤所换来的小人的鄙视，要是他只要用一柄小小的刀子，就可以清算他自己的一生？谁愿意负著这样的重担，在烦劳的生命的压迫下呻吟流汗，倘不是因為惧怕不可知的死后，惧怕那从来不曾有一个旅人回来过的神秘之国，是它迷惑了我们的意志，使我们宁愿忍受目前的磨折，不敢向我们所不知道的痛苦飞去？这样，重重的顾虑使我们全变成了懦夫，决心的赤热的光彩，被审慎的思维盖上了一层灰色，伟大的事业在这一种考虑之下，也会逆流而退，失去了行动的意义。且慢！美丽的奥菲利婭！——女神，在你的祈祷之中，不要忘记替我忏悔我的罪孽。']]

3.Use Amazon Translate for the translation¶
In [5]:
amz_data = []

                                          
for item in data:
    result = translate.translate_text(Text=item[0],  SourceLanguageCode="en", TargetLanguageCode="zh")
    amz_data.append([result.get('TranslatedText'),result.get('TranslatedText')])

amz_data
Out[5]:
[['场景 I. 城堡里的一个房间。', '场景 I. 城堡里的一个房间。'],
 ['输入克劳迪乌斯国王、格特鲁德女王、波洛尼乌斯、奥菲莉亚、罗森克兰茨和吉尔登斯滕',
  '输入克劳迪乌斯国王、格特鲁德女王、波洛尼乌斯、奥菲莉亚、罗森克兰茨和吉尔登斯滕'],
 ['克劳迪乌斯国王', '克劳迪乌斯国王'],
 ['而且你能不能在任何情况下从他那里得知为什么他会陷入这种困惑, 在动荡而危险的疯狂中，他所有的安静日子都如此残酷地磨碎?',
  '而且你能不能在任何情况下从他那里得知为什么他会陷入这种困惑, 在动荡而危险的疯狂中，他所有的安静日子都如此残酷地磨碎?'],
 ['ROSENCRANTZ', 'ROSENCRANTZ'],
 ['他确实承认自己感到分心。但是他绝对不会说话的原因。', '他确实承认自己感到分心。但是他绝对不会说话的原因。'],
 ['吉尔登斯滕', '吉尔登斯滕'],
 ['我们也没有发现他向前发出声音, 但是, 狡猾的疯狂, 保持冷漠, 我们什么时候能让他坦白他的真实状态.',
  '我们也没有发现他向前发出声音, 但是, 狡猾的疯狂, 保持冷漠, 我们什么时候能让他坦白他的真实状态.'],
 ['格特鲁德女王', '格特鲁德女王'],
 ['他收到你的好吗？', '他收到你的好吗？'],
 ['哈姆雷特', '哈姆雷特'],
 ['成为，还是不成为：这就是问题所在：是否在头脑中更高尚地遭受痛苦的巨大财富的吊索和箭，还是拿起武器抵御麻烦的海洋，并通过反对结束他们？去死:睡觉; 不再; 睡一觉就说我们结束了心痛和一千次自然冲击那肉是继承者, “这是虔诚的愿望”.死，睡觉；睡觉：有可能做梦：是的，有麻醉；因为在死亡的睡眠中，梦想可能会来临当我们洗掉这个致命的线圈时，必须让我们停下来：有尊重使这么长寿命的灾难；对于谁会承受时间的鞭子和轻蔑，压迫者错了，骄傲的人继续, 鄙视的爱情的痛苦, 法律的拖延, 办公室的傲慢和拒绝那些不值得的人的耐心优点, 他自己什么时候可以安静地用裸露的身体做出安静?Fardels 会忍受谁, 在疲惫的生活中咕 gr 和流汗, 但是死后对某事的恐惧, 没有旅行者从那里回来的未发现的国家, 困惑意志，让我们宁愿忍受自己拥有的那些弊病，而不是飞向我们不认识的其他人?因此，良心确实使我们所有人胆怯。因此，分辨率的本机色调被苍白的思想所吓倒了，还有伟大的时刻和伟大的企业在这方面，他们的潮流变得错了，而且失去了行动的名字。— 现在软你！公平的奥菲莉亚！若虫，在你的监狱里记住我所有的罪过。',
  '成为，还是不成为：这就是问题所在：是否在头脑中更高尚地遭受痛苦的巨大财富的吊索和箭，还是拿起武器抵御麻烦的海洋，并通过反对结束他们？去死:睡觉; 不再; 睡一觉就说我们结束了心痛和一千次自然冲击那肉是继承者, “这是虔诚的愿望”.死，睡觉；睡觉：有可能做梦：是的，有麻醉；因为在死亡的睡眠中，梦想可能会来临当我们洗掉这个致命的线圈时，必须让我们停下来：有尊重使这么长寿命的灾难；对于谁会承受时间的鞭子和轻蔑，压迫者错了，骄傲的人继续, 鄙视的爱情的痛苦, 法律的拖延, 办公室的傲慢和拒绝那些不值得的人的耐心优点, 他自己什么时候可以安静地用裸露的身体做出安静?Fardels 会忍受谁, 在疲惫的生活中咕 gr 和流汗, 但是死后对某事的恐惧, 没有旅行者从那里回来的未发现的国家, 困惑意志，让我们宁愿忍受自己拥有的那些弊病，而不是飞向我们不认识的其他人?因此，良心确实使我们所有人胆怯。因此，分辨率的本机色调被苍白的思想所吓倒了，还有伟大的时刻和伟大的企业在这方面，他们的潮流变得错了，而且失去了行动的名字。— 现在软你！公平的奥菲莉亚！若虫，在你的监狱里记住我所有的罪过。']]

4. Use Google Translate to get the query¶
In [6]:
# pip install googletrans
from googletrans import Translator
translator = Translator(service_urls=[ 'translate.google.cn',])
google_data = []
for item in data:
    trans=translator.translate(item[0], src='en', dest='zh-cn')
    google_data.append([trans.origin,trans.text])
google_data
Out[6]:
[['SCENE I. A room in the castle.', '场景我。城堡的一个房间。'],
 ['Enter KING CLAUDIUS, QUEEN GERTRUDE, POLONIUS, OPHELIA, ROSENCRANTZ, and GUILDENSTERN',
  '进入克劳迪斯国王，皇后格鲁鲁德，波隆座，奥蒙翁，罗森克朗兹和吉伦森特'],
 ['KING CLAUDIUS', '克劳迪斯国王'],
 ['And can you, by no drift of circumstance, Get from him why he puts on this confusion, Grating so harshly all his days of quiet With turbulent and dangerous lunacy?',
  '无论如何，没有环境漂移，从他那里得到为什么他陷入了这种混乱，讽刺地抚摸着他的骚乱和危险的疯狂的所有日子？'],
 ['ROSENCRANTZ', 'rosencrantz.'],
 ['He does confess he feels himself distracted; But from what cause he will by no means speak.',
  '他承认他觉得自己分心了;但从什么原因就不会说话。'],
 ['GUILDENSTERN', '瓜塞恩'],
 ['Nor do we find him forward to be sounded, But, with a crafty madness, keeps aloof, When we would bring him on to some confession Of his true state.',
  '我们也没有发现他发出声好，但是，随着狡猾的疯狂，当我们把他带到一些对他真正的国家的忏悔时保持冷漠。'],
 ['QUEEN GERTRUDE', '女王格鲁鲁德'],
 ['Did he receive you well?', '他收到你了吗？'],
 ['HAMLET', '村庄'],
 ["To be, or not to be: that is the question: Whether 'tis nobler in the mind to suffer The slings and arrows of outrageous fortune, Or to take arms against a sea of troubles,And by opposing end them? To die: to sleep; No more; and by a sleep to say we end The heart-ache and the thousand natural shocks That flesh is heir to, 'tis a consummation Devoutly to be wish'd. To die, to sleep; To sleep: perchance to dream: ay, there's the rub; For in that sleep of death what dreams may come When we have shuffled off this mortal coil, Must give us pause: there's the respect That makes calamity of so long life; For who would bear the whips and scorns of time, The oppressor's wrong, the proud man's contumely, The pangs of despised love, the law's delay, The insolence of office and the spurns That patient merit of the unworthy takes, When he himself might his quietus make With a bare bodkin? who would fardels bear, To grunt and sweat under a weary life, But that the dread of something after death, The undiscover'd country from whose bourn No traveller returns, puzzles the will And makes us rather bear those ills we have Than fly to others that we know not of? Thus conscience does make cowards of us all; And thus the native hue of resolution Is sicklied o'er with the pale cast of thought, And enterprises of great pith and moment With this regard their currents turn awry, And lose the name of action.--Soft you now! The fair Ophelia! Nymph, in thy orisons Be all my sins remember'd.",
  '是，还是不成为：这是问题：无论是否铭记的令人沮丧的愤怒和令人愤慨的财富，或者抓住烦恼的海洋，而且通过对手呢？死：睡觉;不再;睡觉说我们结束了心痛和肉体是继承人的一千个自然冲击，“这是一个虔诚地愿意的完善。死，睡觉;睡觉：跋涉到梦想：ay，有摩擦;因为在那个死亡的睡眠中，当我们摆脱这种凡人的灯卷时可能会出现梦想，必须给我们暂停：尊重这么长的寿命会产生灾难;对于谁会抱怨鞭子和蔑视时间，压缩机的错误，骄傲的人的概括，沉浸的爱情的痛苦，法律的延迟，办公室的傲慢和摒弃了患者的不值得的逃避，当他自己可能是他的静胸部用裸体制作？谁会疯狂，疲惫和汗水疲惫，但死亡之后的恐惧，从谁没有旅行者回来的恐惧，归咎于我们的意志，让我们相当耐生，我们却比飞往飞往那些弊病我们不知道的其他人？因此，良心确实使我们所有人的懦夫;因此，决议的原生色调被带着苍白的思想，而伟大的髓中的企业和当时的瞬间，这就是他们的潮流转动，失去了行动的名称.--现在柔软你！公平的ophelia！若虫，在你的奥斯顿中都是我的罪。']]

Machine Translation Quality: BLEU Scores¶
The BLEU score is a string-matching algorithm that provides basic quality metrics of Machine Translation.
The central idea behind BLEU is that the closer a machine translation is to a professional human translation, the better it is.
	•	BLEU (BiLingual Evaluation Understudy) is a metric for automatically evaluating machine-translated text.
	•	BLEU score is a number between zero and one that measures the similarity of the machine-translated text to a set of high quality.
To calculate the bleu score, we first import sentence_bleu method from nltk.translate.bleu. Then we use the split method to split the texts into lists of sentences, and we use the sentence_bleu method to calculate the bleu score of the translated text with the reference text of professional human translation. We will calculate both the bleu score of Amazon Translation and the bleu score of Google Translate and analyze the results.
In [7]:
from nltk.translate.bleu_score import sentence_bleu
from pandas.core.frame import DataFrame
result_google = []
result_amz = []
for i in range(len(data)):
    # orgin ch
    reference = []
    for char in data[i][1]:
        reference.append(char)
        
    # google 
    candidate_google = []
    for char in google_data[i][1]:
        candidate_google.append(char)
        
            
    # amz 
    candidate_amz = []
    for char in amz_data[i][1]:
        candidate_amz.append(char)

    score_google = sentence_bleu(reference, candidate_google)
    result_google.append([reference, candidate_google,score_google])
    
    score_amz = sentence_bleu(reference, candidate_amz)
    result_amz.append([reference, candidate_amz,score_amz])
    
result_amz = DataFrame(result_amz)
result_amz.columns = ['orgin','amzon','score']
result_amz

/home/luis/anaconda3/envs/BioKG/lib/python3.7/site-packages/nltk/translate/bleu_score.py:515: UserWarning: 
The hypothesis contains 0 counts of 2-gram overlaps.
Therefore the BLEU score evaluates to 0, independently of
how many N-gram overlaps of lower order it contains.
Consider using lower n-gram order or use SmoothingFunction()
  warnings.warn(_msg)
/home/luis/anaconda3/envs/BioKG/lib/python3.7/site-packages/nltk/translate/bleu_score.py:515: UserWarning: 
The hypothesis contains 0 counts of 3-gram overlaps.
Therefore the BLEU score evaluates to 0, independently of
how many N-gram overlaps of lower order it contains.
Consider using lower n-gram order or use SmoothingFunction()
  warnings.warn(_msg)
/home/luis/anaconda3/envs/BioKG/lib/python3.7/site-packages/nltk/translate/bleu_score.py:515: UserWarning: 
The hypothesis contains 0 counts of 4-gram overlaps.
Therefore the BLEU score evaluates to 0, independently of
how many N-gram overlaps of lower order it contains.
Consider using lower n-gram order or use SmoothingFunction()
  warnings.warn(_msg)
Out[7]:

orgin
amzon
score
0
[第, 一, 场, , 城, 堡, 中, 一, 室]
[场, 景, , I, ., , 城, 堡, 里, 的, 一, 个, 房, 间, 。]
1.384293e-231
1
[国, 王, 、, 王, 后, 、, 波, 洛, 涅, 斯, 、, 奥, 菲, 利, 婭, ...
[输, 入, 克, 劳, 迪, 乌, 斯, 国, 王, 、, 格, 特, 鲁, 德, 女, ...
1.434713e-231
2
[国, 王]
[克, 劳, 迪, 乌, 斯, 国, 王]
1.331960e-231
3
[你, 们, 不, 能, 用, 迂, 回, 婉, 转, 的, 方, 法, ，, 探, 出, ...
[而, 且, 你, 能, 不, 能, 在, 任, 何, 情, 况, 下, 从, 他, 那, ...
1.334773e-231
4
[罗, 森, 格, 兰, 兹]
[R, O, S, E, N, C, R, A, N, T, Z]
0.000000e+00
5
[他, 承, 认, 他, 自, 己, 有, 些, 神, 经, 迷, 惘, ，, 可, 是, ...
[他, 确, 实, 承, 认, 自, 己, 感, 到, 分, 心, 。, 但, 是, 他, ...
1.448850e-231
6
[吉, 尔, 登, 斯, 吞]
[吉, 尔, 登, 斯, 滕]
1.722982e-231
7
[他, 也, 不, 肯, 虚, 心, 接, 受, 我, 们, 的, 探, 问, ；, 当, ...
[我, 们, 也, 没, 有, 发, 现, 他, 向, 前, 发, 出, 声, 音, ,, ...
1.180800e-231
8
[王, 后]
[格, 特, 鲁, 德, 女, 王]
1.164047e-231
9
[他, 对, 待, 你, 们, 还, 客, 气, 吗, ？]
[他, 收, 到, 你, 的, 好, 吗, ？]
1.531972e-231
10
[哈, 姆, 莱, 特]
[哈, 姆, 雷, 特]
1.695406e-231
11
[生, 存, 还, 是, 毁, 灭, ，, 这, 是, 一, 个, 值, 得, 考, 虑, ...
[成, 为, ，, 还, 是, 不, 成, 为, ：, 这, 就, 是, 问, 题, 所, ...
1.304989e-231
In [8]:
result_google = DataFrame(result_google)
result_google.columns = ['orgin','google','score']
result_google
Out[8]:

orgin
google
score
0
[第, 一, 场, , 城, 堡, 中, 一, 室]
[场, 景, 我, 。, 城, 堡, 的, 一, 个, 房, 间, 。]
1.384293e-231
1
[国, 王, 、, 王, 后, 、, 波, 洛, 涅, 斯, 、, 奥, 菲, 利, 婭, ...
[进, 入, 克, 劳, 迪, 斯, 国, 王, ，, 皇, 后, 格, 鲁, 鲁, 德, ...
1.374000e-231
2
[国, 王]
[克, 劳, 迪, 斯, 国, 王]
1.384293e-231
3
[你, 们, 不, 能, 用, 迂, 回, 婉, 转, 的, 方, 法, ，, 探, 出, ...
[无, 论, 如, 何, ，, 没, 有, 环, 境, 漂, 移, ，, 从, 他, 那, ...
1.262708e-231
4
[罗, 森, 格, 兰, 兹]
[r, o, s, e, n, c, r, a, n, t, z, .]
0.000000e+00
5
[他, 承, 认, 他, 自, 己, 有, 些, 神, 经, 迷, 惘, ，, 可, 是, ...
[他, 承, 认, 他, 觉, 得, 自, 己, 分, 心, 了, ;, 但, 从, 什, ...
1.499007e-231
6
[吉, 尔, 登, 斯, 吞]
[瓜, 塞, 恩]
0.000000e+00
7
[他, 也, 不, 肯, 虚, 心, 接, 受, 我, 们, 的, 探, 问, ；, 当, ...
[我, 们, 也, 没, 有, 发, 现, 他, 发, 出, 声, 好, ，, 但, 是, ...
1.307510e-231
8
[王, 后]
[女, 王, 格, 鲁, 鲁, 德]
1.164047e-231
9
[他, 对, 待, 你, 们, 还, 客, 气, 吗, ？]
[他, 收, 到, 你, 了, 吗, ？]
1.583977e-231
10
[哈, 姆, 莱, 特]
[村, 庄]
0.000000e+00
11
[生, 存, 还, 是, 毁, 灭, ，, 这, 是, 一, 个, 值, 得, 考, 虑, ...
[是, ，, 还, 是, 不, 成, 为, ：, 这, 是, 问, 题, ：, 无, 论, ...
1.310295e-231

Conclusion¶
When amazon is used as the machine translation server, the results are better than Google Translate. However, the performance of proper nouns, context and slang is still inadequate. Neural network machine translation will be widely adopted at a faster pace in the future, as the volume of content requiring localization explodes. The COVID-19 pandemic has accelerated the digital transformation of many businesses and created more demand for translation services. At the same time, content needs to be more targeted and diverse. These market conditions make it easier to localize certain content using machine translation, sometimes without human translation supervision.
In [ ]:
 
