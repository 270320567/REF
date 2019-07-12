from sklearn.feature_selection import RFE,RFECV
from sklearn.linear_model import LinearRegression,Ridge,Lasso,ElasticNet
import numpy as np
import pandas as pd
from scipy.stats import pearsonr
from sklearn.model_selection import KFold,StratifiedKFold,GridSearchCV,RandomizedSearchCV
from scipy.stats import uniform
from matplotlib import cm
from matplotlib import pyplot as plt
from matplotlib.ticker import LinearLocator, FormatStrFormatter
from mpl_toolkits.mplot3d import Axes3D
  
data = pd.read_csv(r'C:\Users\win10\Desktop\bishi\data.csv', header=0) #header=0表示

'''
#尝试通过translate处理特殊字符，问题是原来的nan不能dropna，新的NaN可以dropna
data = data.dropna(thresh=285)
data= data.dropna(axis=1,thresh=1200)
print(data.shape)
def  remove_char(x):
    x = str(x)
    remove1 = x.maketrans('','',',N-+%#$^&* ')
    x=x.translate(remove1)
    remove2 = x.maketrans(',','.')
    x=x.translate(remove2)
    return x
#data_new = data.applymap(remove_char)    
'''
#
data_new = data.replace('-',np.nan)
data_new = data_new.replace('N',np.nan)
data_new = data_new.replace('3,500+','3500')
data_new = data_new.replace('250,000+','250000')
'''
#尝试通过循环方式处理字特殊字符
def float_char(x):
    try:
        x = float(x)
    except:
        x = np.nan
    return x
#data_new = data.applymap(float_char)
nan_all = data_new.isnull()
nan_col1 = data_new.isnull().any() 
nan_col2 = data_new.isnull().all() 
'''
data_new = data_new.dropna(thresh=285)
data_new = data_new.dropna(axis=1,thresh=1200)
data_new2 = pd.DataFrame(data_new,dtype='float')
#填充缺失值
#print(data_new2.isnull())
#data_new3 = data_new2.fillna(method='ffill',axis=1)
for column in data_new2.columns:
    data_new2[column].fillna(data_new2[column].mean(),inplace=True) 
#print(data_new2.isnull())


#定义变量
label = np.array(data_new2)[:, -1]
feature = np.array(data_new2)[:, :-1]
print(label)
print(feature)

#特征选择
rfr = Ridge(alpha=10000, fit_intercept=True, normalize=True,
                  copy_X=True, max_iter=3000, tol=0.00001, solver='auto')

#rfr = ElasticNet(alpha=100000,l1_ratio=0.5,fit_intercept=True,normalize=True,
#                 precompute=True,max_iter=100000,copy_X=True,warm_start=False,
#                 positive=False,random_state=None,selection='cyclic',)
#rfr = LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
#                       normalize=True)
rfecv = RFECV(estimator=rfr,step=1,min_features_to_select=1, 
              cv=3, scoring='r2')
rfecv.fit(feature, label)

#查看结果
print('特征数量：%.3f' %rfecv.n_features_)
#print(rfecv.ranking_)
#print(rfecv.support_)
print(np.shape(rfecv.grid_scores_)) 
#print(rfecv.score(feature, label))
print('回归器：%s' %rfecv.estimator_ )
plt.xlabel("Number of features selected")
plt.ylabel("R2 score")
plt.plot(range(1, len(rfecv.grid_scores_) + 1), rfecv.grid_scores_)
plt.show()

#重新设定变量集
data_new3 =data_new2.iloc[:,rfecv.support_]
data_new3['Label'] = label
#print(data_new3)
label_new = np.array(data_new3)[:, -1]
feature_new = np.array(data_new3)[:, :-1]
 
#调参    
rfr2 =Ridge(fit_intercept=True, normalize=True,
                  copy_X=True,  tol=1e-4, solver='auto')
alpht_list = list(range(1,100000,1000))
max_iter_list = list(range(100,10000,100))
param_grid = {'alpha':alpht_list,'max_iter':max_iter_list}
#param_grid = {'alpha':uniform(),'max_iter':uniform()}
grid = GridSearchCV(estimator=rfr2,param_grid=param_grid,scoring='r2',cv=3)
#grid = RandomizedSearchCV(estimator=rfe,param_distributions=param_grid,
#                          n_iter=100,scoring='r2',random_state=7)
grid.fit(feature_new, label_new)

#查看结果
print('最高得分：%.3f' %grid.best_score_)
print('最优参数：%s' %grid.best_estimator_.alpha)
print('最优参数：%s' %grid.best_estimator_.max_iter)
print('回归器：%s' %grid.estimator)
print(grid.cv_results_['std_test_score'])

#画图
x = np.array(alpht_list)
y = np.array(max_iter_list)
z = grid.cv_results_['mean_test_score'].reshape(len(max_iter_list),len(alpht_list))
ax = plt.figure().gca(projection='3d')
X, Y = np.meshgrid(x,y)
print(np.shape((X,Y)))
ax.set_xlabel('alpha')
ax.set_ylabel('max_iter')
ax.set_zlabel('R2 score')
fig_3d = ax.plot_wireframe(X, Y, z,rstride=10, cstride=10)
surf = ax.plot_surface(X, Y, z, cmap=cm.coolwarm,
                       linewidth=0, antialiased=False)
plt.show()


'''
#尝试通过定义函数画三维图
def zz(x,y):    
    rfr2 =Ridge( fit_intercept=True, normalize=True,
                  copy_X=True, tol=1e-4, solver='auto')
    param_grid = {'alpha':x, 'max_iter':y}
    grid = GridSearchCV(estimator=rfr2,param_grid=param_grid,scoring='r2',cv=3)
    grid.fit(feature_new, label_new)     
    z=list(grid.cv_results_['std_test_score'])
    return z'''

'''
#尝试通过循环方式调参并画图
ref_ele = np.empty((1,3))
for alpha in (range(1,10000,1000)):
    for max_iter in range(100,10000,1000):
        
        rfr2 =ElasticNet(alpha=1000,l1_ratio=0.5,fit_intercept=True,
                         normalize=False,precompute=True,max_iter=2000,copy_X=True,warm_start=False,
                 positive=False,random_state=None,selection='cyclic',)
        rfr2.fit(feature_new, label_new)
        score = rfr2.score(feature_new, label_new)
        ref_ele = np.row_stack((ref_ele,np.array([alpha, max_iter,score])))
print(ref_ele ) 
x = ref_ele[1:,0]
y = ref_ele[1:,1]
z = ref_ele[1:,2].reshape(len(y),len(x))
X, Y = np.meshgrid(x,y)

ax = plt.figure().gca(projection='3d')
surf = ax.plot_surface(X, Y, z, cmap=cm.coolwarm,
                       linewidth=0, antialiased=False)
fig.colorbar(surf, shrink=0.5, aspect=5)
plt.show()
#nan_result.to_csv('test.csv',index=False,sep=',')
'''
