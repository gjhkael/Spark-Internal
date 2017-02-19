# RDD

##RDD是什么

RDD的全名是Resilient Distributed Dataset，意思是容错的分布式数据集，每一个RDD都会有5个特征：

    1、有一个分片列表。就是能被切分，和hadoop一样的，能够切分的数据才能并行计算。
        2、有一个函数计算每一个分片，这里指的是下面会提到的compute函数。
	    3、对其他的RDD的依赖列表，依赖还具体分为宽依赖和窄依赖，但并不是所有的RDD都有依赖。
	        4、可选：key-value型的RDD是根据哈希来分区的，类似于mapreduce当中的Paritioner接口，控制key分到哪个reduce。
		    5、可选：每一个分片的优先计算位置（preferred locations），比如HDFS的block的所在位置应该是优先计算的位置。
		    要是每个RDD都可以把上述的5个特征搞清楚，那么RDD也就搞的很透彻了，简单的说，RDD就是一个抽象类，上面有各种实现，每一种实现，就是一个RDD，
		    RDD可以转换成另外一种RDD（前提是可以转换），简单的说，RDD其实封装了数据的源头、数据如何计算、计算完返回上面结果。RDD根据功能可以分为：
		    transformation和action两大类。

		    ##RDD分类

		    | Transformation | Meaning |
		    |:-----------|:-------------|
		    | map(func) | 对每个元素进行func的计算，然后返回一个新的RDD |
		    | flatMap(func) | 对每个元素进行func的计算，然后返回一个新的RDD 与map的不同之处是返回的计算结果不一样 |
		    | filter(func) | 对每个元素进行func的计算，结果为true的元素得到保存，然后返回一个新的RDD |
		    | mapPartitions(func) | 和map类似，但是它是对partition为单位进行计算 |
		    | mapPartitionsWithIndex(func) |和mapPartitions类似，但是对每个partition进行了索引 |
		    | map(func) | 对每个元素进行func的计算，然后返回一个新的RDD |

		    ##主要内容
