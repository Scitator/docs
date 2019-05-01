---
title: wandb.log
sidebar_label: wandb.log
---

## Overview

Calling `wandb.log(dict)` logs the keys and values of the dictionary passed in and associates the values with a _step_. Wandb.log can log histograms and custom matplotlib objects and rich media.

`wandb.log(dict)` accepts a few keyword arguments:

- **step** &mdash; Step to associate the log with (see [Incremental Logging](#incremental-logging))
- **commit** &mdash; If true, increments the step associated with the log(_default: true_)

## Example

Any time you call `wandb.log` and pass in a dictionary of keys and values, it will be saved as a new time step for plots in the W&B app.

```python
wandb.log({'accuracy': 0.9, 'epoch': 5})
```

## Incremental Logging

If you want to log to a single history step from lots of different places in your code you can pass a step index to `wandb.log()` as follows:

```python
wandb.log({'loss': 0.2}, step=step)
```

As long as you keep passing the same value for `step`, W&B will collect the keys and values from each call in one unified dictionary. As soon you call `wandb.log()` with a different value for `step` than the previous one, W&B will write all the collected keys and values to the history, and start collection over again. Note that this means you should only use this with consecutive values for `step`: 0, 1, 2, .... This feature doesn't let you write to absolutely any history step that you'd like, only the "current" one and the "next" one.

You can also set **commit=False** in `wandb.log` to accumulate metrics, just be sure to call `wandb.log` without the **commit** flag to persist the metrics.

```python
wandb.log({'loss': 0.2}, commit=False)
# Somewhere else when I'm ready to report this step:
wandb.log({'accuracy': 0.8})
```

## Logging Tensors

If you pass a numpy array, pytorch tensor or tensorflow tensor to `wandb.log` we automatically convert it as follows:

1. If the object has a size of 1 just log the scalar value
2. If the object has a size of 32 or less, convert the tensor to json
3. If the object has a size greater than 32, log a histogram of the tensor

### Logging Plots

```python
import matplotlib.pyplot as plt
plt.plot([1, 2, 3, 4])
plt.ylabel('some interesting numbers')
wandb.log({"chart": plt})
```

You can pass a `matplotlib` pyplot or figure object into `wandb.log`. By default we'll convert the plot into a [plotly](https://plot.ly) plot. If you want to explictly log the plot as an image, you can pass the plot into `wandb.Image`. We also accept directly logging [plotly](https://plot.ly/) charts.

### Logging Images

```python
wandb.log({"examples": [wandb.Image(numpy_array_or_pil, caption="Label")]})
```

If a numpy array is supplied we assume it's gray scale if the last dimension is 1, RGB if it's 3, and RGBA if it's 4. If the array contains floats we convert them to ints between 0 and 255. You can specify a [mode](https://pillow.readthedocs.io/en/3.1.x/handbook/concepts.html#concept-modes)
manually or just supply a `PIL.Image`. We recommend you don't add more than 20-50 images per step.

On the W&B runs page, you should edit your graphs and choose "Image Viewer" to see your training images.

### Logging Audio

```python
wandb.log({"examples": [wandb.Audio(numpy_array, caption="Nice", sample_rate=32)]})
```

The maximum number of audio clips that can be logged per step is 100.

### Logging Text / Tables

```python
# Method 1
data = [["I love my phone", "1", "1"],["My phone sucks", "0", "-1"]]
wandb.log({"examples": wandb.Table(data=data, columns=["Text", "Predicted Label", "True Label"])})

# Method 2
table = wandb.Table(columns=["Text", "Predicted Label", "True Label"])})
table.add_data("I love my phone", "1", "1")
table.add_data("My phone sucks", "0", "-1")
wandb.log({"examples": table})
```

By default, the column headers are `["Input", "Output", "Expected"]`. The maximum number of rows is 300.

### Logging HTML

```python
wandb.log({"custom_file": wandb.Html(open("some.html"))})
wandb.log({"custom_string": wandb.Html('<a href="https://mysite">Link</a>')})
```

Custom html can be logged at any key, this exposes an HTML panel on the run page. By default we inject default styles, you can disable default styles by passing `styles=False`.

### Logging Histograms

```python
wandb.log({"gradients": wandb.Histogram(numpy_array_or_sequence)})
wandb.run.summary.update({"gradients": wandb.Histogram(np_histogram=np.histogram(data))})
```

If a sequence is provided as the first argument, we will bin the histogram automatically. You can also pass what is returned from `np.histogram` to the **np_histogram** keyword argument to do your own binning. The maximum number of bins supported is 512. You can use the optional **num_bins** keyword argument when passing a sequence to override the default of 64 bins.

If histograms are in your summary they will appear as sparklines on the individual run pages. If they are in your history, we plot a heatmap of bins over time.


### Logging 3D Objects
```python
wandb.log({"generated_samples":  
           [wandb.Object3D(open("sample_0.obj")),
            wandb.Object3D(open("sample_0.gltf")),
            wandb.Object3D(open("sample_0.glb"))]})
```

Wandb supports logging 3D file types of in three different formats: glTF, glb, obj. The 3D files will be viewable on the run page upon completion of your run. 

### Logging Point Clouds

```python
point_cloud = np.array([[0, 0, 0, COLOR...], ...])

wandb.log({"point_cloud": wandb.Object3D(point_cloud)})
```

Numpy arrays logged via wandb.Object3D will be rendered as 3D point clouds.

Supported numpy shapes include three different color schemes:
 * `[[x y z],       ...]` nx3
 * `[[x y z c],     ...]` nx4 | c is a category with supported range [1, 14](Useful for segmentation)
 * `[[x y z r g b], ...]` nx6 | r,g,b are values in the range [0,255] for Red, Green, Blue color channels.


## Summary Metrics

The summary statistics are used to track single metrics per model. If a summary metric is modified, only the updated state is saved. We automatically set summary to the last history row added unless you modify it manually. If you change a summary metric, we only persist the last value it was set to.

```python
wandb.init(config=args)

best_accuracy = 0
for epoch in range(1, args.epochs + 1):
  test_loss, test_accuracy = test()
  if (test_accuracy > best_accuracy):
    wandb.run.summary["best_accuracy"] = test_accuracy
    best_accuracy = test_accuracy
```

You may want to store evaluation metrics in a runs summary after training has completed. Summary can handle numpy arrays, pytorch tensors or tensorflow tensors. When a value is one of these types we persist the entire tensor in a binary file and store high level metrics in the summary object such as min, mean, variance, 95% percentile, etc.

```python
api = wandb.Api()
run = api.run("username/project/run_id")
run.summary["tensor"] = np.random.random(1000)
run.summary.update()
```

## Accessing Logs Directly

The history object is used to track metrics logged by _wandb.log_. You can access a mutable dictionary of metrics via `run.history.row`. The row will be saved and a new row created when `run.history.add` or `wandb.log` is called.

### Tensorflow Example

```python--tensorflow
wandb.init(config=flags.FLAGS)

# Start tensorflow training
with tf.Session() as sess:
  sess.run(init)

  for step in range(1, run.config.num_steps+1):
      batch_x, batch_y = mnist.train.next_batch(run.config.batch_size)
      # Run optimization op (backprop)
      sess.run(train_op, feed_dict={X: batch_x, Y: batch_y})
      # Calculate batch loss and accuracy
      loss, acc = sess.run([loss_op, accuracy], feed_dict={X: batch_x, Y: batch_y})

      wandb.log({'acc': acc, 'loss':loss}) # log accuracy and loss
```

### PyTorch Example

```python--pytorch
# Start pytorch training
wandb.init(config=args)

for epoch in range(1, args.epochs + 1):
  train_loss = train(epoch)
  test_loss, test_accuracy = test()

  torch.save(model.state_dict(), 'model')

  wandb.log({"loss": train_loss, "val_loss": test_loss})
```
