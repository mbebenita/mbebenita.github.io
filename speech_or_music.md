# Speech or Music? - June 19, 2017

[Opus](https://opus-codec.org/) is a highly versatile audio codec.
In fact, it's comprised of two codecs: Skype’s SILK codec which works well for speech and Xiph.Org’s CELT codec which is best suited for music. Opus picks which codec to use by analyzing the input signal. If the signal sounds like music it uses CELT, if it sounds like speech then it uses SILK, if there's no signal (silence, or noise) then it just uses whatever it was using previously. Opus make this choice every 20 ms (or the typical length of an audio frame.)

This sounds like a fun, and fairly contained problem. A great opportunity to learn about signal processing and machine learning. Opus currently ships with a simple neural network that does this classification (a single hidden layer of 15 neurons.) Can I do better? Can you? I invite you to try.

I should add that I know very little about signal processing, or machine learning for that matter, and that I'm writing things down as I'm figuring them out. I'm more interested in capturing my mistakes than offering a solution.


## Data

The data set is comprised of 1,356,049 frames, or about 7 1/2 hours of speech and music.
It was built by concatenating a large number of files together, and preserving the order of frames in the data set. This is an important detail because it means Opus can use information it gathered from previous frames to classify the current frame. For example, if the previous 10 frames are music, the current frame is most likely also music, even if it is hard to tell by looking at just one frame.

Each frame of audio is broken down into 25 features. In Opus, the feature extraction happens in the [tonality_analysis](https://github.com/xiph/opus/blob/master/src/analysis.c#L315) function. Although it's a bit hard to tell, according to Jean-Marc this code effectively computes the [Cepstrum](https://en.wikipedia.org/wiki/Cepstrum) coefficients.

> TODO: Understand why Cepstrum is used here, and if all the 25 features are the Cepstrum or there are other features thrown in in there.

## Accuracy of current Opus Classifier

I hacked Opus to print out the computed features for each frame and then I took the original classification label (+1 = Music, -1 = Speech, or 0 = Doesn't Matter.) and created [one large text file](//).

| 0 | 1 | 3 | ...  | 25 | 26 |
|:--|:--|:--|:--|:--|:--|
| -4.046364 | -2.284566 | 2.170472 | ...  | +1.0 | ...  |
| -2.632245 |  2.664234 | -2.41234 | ...  | -1.0 | ...  |
| -0.110010 |  3.122312 | 0.154512 | ...  |  0.0 | ...  |

This is in effect all the data we need to train a model on. I also hacked Opus to print out the prediction label. Comparing this with the original label, it looks like Opus currently is 99.14% accurate. So, we need to do better than that.

## Linear Regression

A simple linear regression model for instantaneous classification yields 91% accuracy. By "
instantaneous classification" I mean that the model classifies a frame by looking at it in isolation.

```python
import tensorflow as tf
```

Let's set up a data input pipeline. The *prepare_input* function sets up the input node graph in TensorFlow graph. You can read more details on how this works [here](https://www.tensorflow.org/programmers_guide/reading_data).

```python
def prepare_input(sess, files, batch_size=10000):
  filename_queue = tf.train.string_input_producer(files)
  tf.train.start_queue_runners(sess=sess)
  reader = tf.TextLineReader()
  key, value = reader.read_up_to(filename_queue, batch_size)
  csv = tf.decode_csv(value, record_defaults=[
      [0.0] for x in range(0, 29)], field_delim=' ')
  # Features
  x = tf.stack(csv[0:25], axis=1)
  # Labels: +1 = Music, -1 = Speech, 0 = Don't Care. This converts the label
  # into a one hot vector which makes it easier to work with.
  #   -1 -> <1, 0, 0>
  #    0 -> <0, 1, 0>
  #   +1 -> <0, 0, 1>
  Y = tf.one_hot(tf.cast(csv[27:28], tf.int32) + 1, 3)[0]
  return x, Y
```

Note that the function above only creates a graph, it doesn't actually read or parse any data. To do that you'll have to "run" its nodes. In TensorFlow, the general programming model is that create and wire nodes in a computational graph, then you create sessions in which you evaluate those nodes. You don't have to evaluate the entire graph at once, sometimes you can evaluate parts of it, copy the results from one part of the graph into another through placeholders. TensorFlow works best if you create large graphs and let it figure out how to distribute the load across multiple threads or GPUs (but I digress.)

```python
# Create a TensorFlow session.
sess = tf.Session()

# Create input graph.
x_input, Y_input = prepare_input(sess, ["train.csv"], 10000)

# Run input graph.
x_batch, Y_batch = sess.run([x_input, Y_input])

print(x_batch, Y_batch)
```

Now that we've got some data to load let's train a model.

```python
def run_linear_regression():
  config = tf.ConfigProto(intra_op_parallelism_threads=8,
                          inter_op_parallelism_threads=8,
                          allow_soft_placement=True)
  # Create a session.
  sess = tf.Session(config=config)

  # Create a placeholder for the feature tensor.
  x = tf.placeholder(tf.float32, shape=(None, 25), name="x")

  # Create a placeholder for the training labels.
  Y = tf.placeholder(tf.float32, shape=(None, 3), name="Y")

  # Model weights.
  W = tf.Variable(tf.zeros([25, 3]), name="W")

  # Model bias.
  b = tf.Variable(tf.zeros([3]), name="b")

  # Model
  y = tf.matmul(x, W) + b

  # Optimization function.
  cross_entropy = tf.reduce_mean(
    tf.nn.softmax_cross_entropy_with_logits(labels=Y, logits=y))

  # Training step.
  step = tf.train.GradientDescentOptimizer(0.01).minimize(cross_entropy)

  # Have we predicted correctly?
  correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(Y, 1))

  # Special case of our problem, we don't care what our prediction is if the original
  # label is "Don't Care"
  correct_prediction = tf.logical_or(correct_prediction, tf.equal(tf.argmax(Y, 1), 1))

  # What percentage of predictions are correct?
  percent_correct = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

  # Wipe out the variables we're about to train.
  sess.run(tf.global_variables_initializer())

  # Create the input subgraph.
  x_input, Y_input = prepare_input(sess, [SPEECH_MUSIC_TRAIN], 10000)

  for i in range(100):
    # Read a batch of 10,000 records from the training set.
    x_batch, Y_batch = sess.run([x_input, Y_input])

    # Run a training step with the above loaded records in the placeholders.
    _, crt = sess.run([step, percent_correct], feed_dict={
        x: x_batch, Y: Y_batch})

    # How are we doing?
    print('Train {0}: Accuracy: {1:.0f}%'.format(i, crt * 100))

  # Create another input subgraph but this time load from a different data set.
  x_input, Y_input = prepare_input(sess, [SPEECH_MUSIC_TEST], 10000)

  for i in range(100):
    # Read a batch of 10,000 records from the validation set.
    x_batch, Y_batch = sess.run([x_input, Y_input])

    # Check the accuracy of the above loaded records in the placeholders.
    _, crt = sess.run([step, percent_correct], feed_dict={
        x: x_batch, Y: Y_batch})

    # How are we doing?
    print('Test {0}: Accuracy: {1:.0f}%'.format(i, crt * 100))
```

When I first tried this out, I connected the model directly to the input graph, but then I couldn't figure out how to have the inputs nodes read from the validation set. The easiest solution was to disconnect the two input graphs from the model and then use Python to pull data from the input graph, put it in the placeholders and then run the training and validation graphs.












<!--

## Data

Each data record represents a 20ms audio frame. Each frame has 25 extracted [Cepstrum](https://en.wikipedia.org/wiki/Cepstrum) coefficients and a classification label: +1 = Music, -1 = Speech, or 0 = Doesn't Matter.

| 0 | 1 | 3 | ...  | 25 | 26 |
|:--|:--|:--|:--|:--|:--|
| -4.046364 | -2.284566 | 2.170472 | ...  | +1.0 | ...  |
| -2.632245 |  2.664234 | -2.41234 | ...  | -1.0 | ...  |

[Training Set](https://people.xiph.org/~mbebenita/misc/train.gz)

678,024 Records

[Validation Set](https://people.xiph.org/~mbebenita/misc/validate.gz)

339,012 Records

[Test Set](https://people.xiph.org/~mbebenita/misc/test.gz)

339,012 Records-->