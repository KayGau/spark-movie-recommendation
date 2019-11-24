
##�����Ƽ��㷨
###�Ƽ��㷨�ܹ���

![mahua](mahua-logo.jpg)

���Ƽ�ϵͳ�ļܹ�����ͼ��ʾ���ֱ��Ϊ�������������Ƽ��߼����Ƽ����Ρ��������Ƽ����������֣����Ľ�����⼸������Ϊ˳�������ϸ���ܣ����ж�Ӧ�ڴ���Ҳ�ᴩ���������Խ��͡���ϵͳ������python��sparkʵ�ֵģ�������spark��python�µ�רҵ��pyspark��ʵ�֡�

##3.1 ��������
����Ҫ���û�����ʽ��Ϊ�͵�Ӱ����ʽ����Ϊģ�͵���Ҫ����Դ��Movielens2M,������ݰ�����2,000,000���û�����Ϊ��¼��������ǰ�����ǽ��û���������Ϊ������Ϊ��ʽ��Ϊ����Ϊ�ⲻ���û�����ȥ��д�ģ�����ϵͳ��¼�µ���Ϊ�û���Ϊ���ݣ������Ƽ�ϵͳ�ľ�����������������û���ʽ��Ϊ���ݶ�չ���ġ����ں��ߣ�������Ϊ��Ӱ�Ĳ���Ϣ������������Ӱ���ԣ����絼�ݡ���Ա�ȵȣ�����Щ�����ǵ�Ӱ����ʽ��ǩ��������������ʽ������ͳ����Щ��Ϣ��
##3.2 �Ƽ��߼���ģ�ͣ�
###3.2.1 TF-IDF���ƶ�

��Ӧ���룺item_item_matrix.ipynb �ļ�

���룺��Ӱ����ʽ����

��������Ӱi�����Ƶ�N����Ӱ

�������Ƕ�ȡ��Ӱ����ʽ����������ת��ΪpythonRDD��ʽ�ı�����Ȼ������pyspark�Դ���TF-IDF���ƶȼ��㷽���������ƶȼ��㣬���ҽ��й�һ��������Ҫ����������ʾ��

```javascript
#TF-IDF����
documents = rdd.map(lambda l: l[1].split("|"))
from pyspark.mllib.feature import HashingTF, IDF
hashingTF = HashingTF(numFeatures=1000)
tf = hashingTF.transform(documents)
tf.cache()
idf = IDF().fit(tf)
tfidf = idf.transform(tf)
#��һ������
from pyspark.mllib.feature import Normalizer
labels = rdd.map(lambda l: l[0])
features = tfidf
normalizer = Normalizer()
data = labels.zip(normalizer.transform(features))

```
��Ϊ���ǵ�item����N�ǳ����࣬����N^2��������ݻ����Ĵ�������Դ��ͬʱΪ��ʵʱ�Ƽ�ϵͳ��Ҫ�󣬴���Ѱ�����Ӱi�����Ƶ�N����ӰҲ�ǳ���ʱ����������ϣ��������ʱ�ܹ�����Ѱ�����Ӱi�����Ƶ�N����Ӱ����������ֱ�۵��Դ�Ϊ���صĽ����������������python�Դ���ϡ�����ȥ�����Ӱ�͵�Ӱ�����ƶȣ������������Ӱi�����Ƶ�N����Ӱ����Ϊ�ļ���Ϊitem_similarity.pkl

```javascript
from scipy import sparse
import copy
coo = sparse.load_npz('./tfidf.npz')
lil = coo.tolil()
n_user,n_item = lil.shape
from tqdm import tqdm
import numpy as np
import copy
batch = 100
lil_right = lil.transpose()
topk = []
for i in tqdm(range(int(n_user/batch))):
    lil_left = lil[i*batch:(i+1)*batch]
    sim_lil = lil_left.dot(lil_right)  
    v = -sim_lil.toarray()
    top5 = copy.deepcopy(np.argsort(v)[:,:5])
topk.append(top5)
```
###3.2.2 ���־���ֽⷽ��

���룺�û����־���

������û���ϲ����N����Ӱ

��Ӧ���룺RS.ipynb�ļ�

�����Ƽ�ϵͳ��˵�������󳡾�������Ԥ����Top-N�Ƽ�������Ԥ�ⳡ����Ҫ����������վ�������û����Լ������ĵ�Ӱ�����ٷ֣�MovieLens���������û����Լ��������鼮���۶��ٷ֣�Douban�������о���ֽ⼼����ҪӦ���ڸó�����
���ǲ�����pyspark�Դ��ľ���ֽ��Ƽ��㷨���÷����ǻ���NIPS2008��ķ���Probabilistic matrix factorizationΪ��ʧ���������ý�����С���˵ķ�������ģ���Ż�����Ҫ�������������ʾ��

```javascript
from pyspark.ml.recommendation import ALS
#train for model
rec = ALS(maxIter=10, regParam=0.01, userCol='userId', itemCol='movieId_num', ratingCol='rating', nonnegative=True,
                 coldStartStrategy='drop')
rs_model = rec.fit(train_df)
```

������Ҫ����ÿ���û���ϲ����N����Ӱ��λ�������Ϊ�ļ�reommendation1.pkl��Ϊ�����Ƽ�����Ҫ����һֱ֮һ��

###3.2.3 ����Ʒ����ֽ�
���룺�û����־���

���������Ʒ����ƾ���

��Ӧ���룺markov_sim.ipynb.ipynb�ļ�

��ʵ�ϣ��û�����Ϊ�Ƿ���ĳ����Ϊת��ģʽ�ģ�Ϊ�˸��õؽ�ģ�û�����Ϊ������Ƽ�ϵͳ���Ƽ�Ч�������ǽ����Rendle S��WWW2010�Ĺ���Factorizing personalized markov chains for next-basket recommendation��FPMC������ģ�����û���Ϊת��ģʽ��ͬʱΪ��ʹ����Ӧ�����߷��������Ǵ����Ե��ڴ˻�����ֱ�ӷֽ�����Ʒ����ʹ���߷��������߷�������ʵ�֡�
������3.2.2�ж������־���ķֽ⣬���Ƕ�Ӧ�ķֽ�����Ʒ�ת�ƾ������շ��ص�Ӱ֮�������Ʒ����ƾ����������һ�����M_i : [M_j1,��,M_jn]�����Ӧ�ĺ���Ϊ����û������M_i���������п��ܼ������[M_j1,��,M_jn]�еĵ�Ӱ��

##3.3 �Ƽ�����
�����Ƽ����εĲ�ͬ�����ǽ�����Ҫ��Ϊ�����Ƽ������£��������Ƽ������£���ǰ����Ҫ������ѵ��Ϊ����ּ�ڲ�׽�û�������ƫ�ã����������ߣ����뼶�����������ߵ�����ѵ��������Ϊ����ּ�ڲ�׽�û���ƫ�ñ仯��
###3.3.1 �����Ƽ������£�
�����û����־���������Ƽ���3.2.2�����ڴ˲�׸����

���ڵ�Ӱ���ƶȵ������Ƽ���3.2.1��3.2.3����������

���룺��Ӱ�����ƶȣ��û�����Ϊ

������û���ϲ����N����Ӱ

��Ӧ���룺RS2.ipynb�ļ���RS3.ipynb�ļ�

�÷�������Ҫ˼����Ǹ����û���ʷ��Ϊ�ĵ�ӰΪ������ȥѰ����֮���ƣ��������ƣ���Ϊ���ƣ��ĵ�ӰΪ���ݣ���Ϊ�û�����ϲ���ĵ�Ӱ����Ҫ����������ʾ��

```javascript
users_rec = dict()
from tqdm import tqdm
import copy
import collections
values = df[['userId','movieId']].values
recurrent_list = []
for i,line in tqdm(enumerate(values)):
    user,item = line    
    append = copy.deepcopy(item_simpickle[item].tolist())
    if values[i][0] == values[i+1][0]:
        recurrent_list.extend(append)
    else:
        Counter = collections.Counter(recurrent_list)
        rec_user = copy.deepcopy(list(Counter.keys())[::-1][:10])
        users_rec[user]= rec_user
        recurrent_list = []
```

��������CounterȥѰ�ҳ��ִ������ĵ�Ӱ�����շ����û��ڲ�ͬ���ƶ��¿���ϲ����TopK����Ӱ�������recommendation2��recommendation3��ʾ��

###3.3.2 �����Ƽ�
��Ϊ�û�ϣ��ʵʱ����ϵͳ�����������ǵڶ�����ߵȴ������Ӻ��ֵ�ǰ��Ϊ�Ľ������������ϣ������һ�ֺ��뼶�û�ƫ�óأ��Դ�Ϊ���ݽ����Ƽ���ֱ�۵ģ������߼�������ʾ��

![mahua](dynamic.jpg)

��Ȼ���Ƶ�Ӱ��������3.2.1��3.2.3���������õģ�ֻ��O(1)ȥ��ѯ���ɣ�Ȼ����ҪO(1)ȥ�����û�ƫ�óأ�����������������������O(1)ʱ�临�Ӷȵģ����������û���ʵʱ��Ҫ��Ϊ��ʵ��ʵʱ�ԣ��ں��ʵ���Ƽ����ڴ˲��С�
##5.4 ������
��Ϊ������Ա���٣�������ֻ����Ϊһ������ģ����֣����Ǽ򻯵ľ���ʵ����һ�����ӵĹ��̣�������ҽ���������������е��Ƽ��߼����صĽ�������Ƽ���
##5.5 Top-K�Ƽ�
�����������Ľ���Ƽ�ǰN���û�����ϲ���ĵ�Ӱ��
