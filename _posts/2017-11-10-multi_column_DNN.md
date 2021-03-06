---
title: Multi-Column Deep Neural Network
updated: 2017-11-10 23:35
layout: post
author: Wonik Jang
category: DeepLearning_Supervised_classification_MultiColumnDNN
tags:
- Classification
- Multi Column
- Convolutional Neural Network
- Deep Neural Network
- Python
- Tensorflow

---

# **Motivation of implementing MCDNN for Image classification** 
The thing I would like to remind myself is that most of real world data is totally different from MNIST or CIFAR10 in terms of standardization. Suppose your images are generated by a sensor machine through automatic object detection. Since the machine captures an object with a certain degree of freedom, the size of an image, location of an object, and lightness of background might be variant across time and classes. Even the camera and scope of an object are fixed, diverse image pre-processing can affect the model accuracy. In addition, weighted voting or averaging different CNN models has possibility of improving the model performance. As a way to merge such possibilities, [Ciresan etal 2012, CVPR][Ciresan's_paper-] Multi-Column Deep Neural Network (MCDNN), which is an ensemble of CNNs can be applied. This paper argues that combining multiple CNNs and averaging the output score can boost the prediction accuracy with low error rate.

[Ciresan's_paper-]:people.idsia.ch/~juergen/cvpr2012.pdf


![mcdcnn](/result_images/mcdnn.png  "mcdnn")

Illustration image from Ciresan etal 2012 CVPR



# **Project issues**
Image data in one of my multi-classification task has high complexity with overlapping features across classes, which makes it difficult to draw decision borders with a single combination of pre-processing and model, so I applied main concept of MDCNN on multi-classification within the limit of GPU capability(GeForce 1080Ti 11G). Below, I posted a class object that I used to generate multiple graphs and sessions for the training and testing by python & Tensorflow.



# **Class for declaring multiple graphs and sessions**

{% highlight ruby %}

# Initial Set Up

config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.8

param_path = '/path/to/saved/parameter'


NN = 24

class graph_sess(object):

    def __init__(self, N, param_path):

        self.N = N
        self.param_path = param_path

    def graph(self):

        d = collections.OrderedDict()

        for x in range(N):
            if x < 10:
                d["g0{0}".format(x)] = tf.Graph()
            else:
                d["g{0}".format(x)] = tf.Graph()

        return list(d.items())

    def session(self):

        item_res = self.graph()

        for i in range(self.N):

            with item_res[i][1].as_default():

                if i < 10:
                    index = "0{0}".format(i)
                else:
                    index = "{0}".format(i)

                globals()['sess%s' % index] = tf.Session(config=config)

                globals()['saver%s' % index] = tf.train.import_meta_graph(self.param_path + 'model.meta')
                globals()['saver%s' % index].restore(globals()['sess%s' % index], self.param_path + 'model')

                tf.get_default_graph().as_graph_def()

                globals()['x%s' % index] = globals()['sess%s' % index].graph.get_tensor_by_name("input:0")
                globals()['y%s' % index] = globals()['sess%s' % index].graph.get_tensor_by_name("output:0")

                # Depending on the CNN structure, change the dropout or other options.
                globals()['kp1%s' % index] = globals()['sess%s' % index].graph.get_tensor_by_name("keep_prob1:0")
                globals()['kp2%s' % index] = globals()['sess%s' % index].graph.get_tensor_by_name("keep_prob2:0")
                globals()['kp3%s' % index] = globals()['sess%s' % index].graph.get_tensor_by_name("keep_prob3:0")
                globals()['ys%s' % index] = tf.nn.softmax(globals()['y%s' % index])
                globals()['cr%s' % index] = tf.argmax(globals()['ys%s' % index], 1)




graph_sess_inst = graph_sess(NN, param_path)
graph_sess_inst.session()

{% endhighlight %}


# **Wrap-up**
There are sill several issues regarding how to implement combinations of pre-processing and CNN Models. For instance, image pre-processing can have different options like inner padding, zero-padding, and edge filter. Also, the architecture of CNNs can be various through the number of hidden layer, kernel size, and the existence of pooling layer. Lastly, different ways of merging multiple outputs should be considered. Basically, we are able to average outcomes as in the paper or weighted voting. Attacking theses issues will be adjusted by data and project goals.
