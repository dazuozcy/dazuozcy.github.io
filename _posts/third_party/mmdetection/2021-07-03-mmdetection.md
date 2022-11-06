---
layout: post
title: "mmdetection"
author: dazuo
date: 2020-07-03 20:19:00 +0800
categories: [CV]
tags: [mmdetection]
math: true
mermaid: true
---

[官方文档](https://mmdetection.readthedocs.io/zh_CN/latest/1_exist_data_model.html)

```shell
# Train
python tools/train.py configs/yolox/yolox_tiny_8x8_300e_coco.py

# Test
python tools/test.py configs/yolox/yolox_tiny_8x8_300e_coco.py xxx.pth --eval mAP
```



# Train

Traceback

```shell
File "./tools/train.py", line 189 in <module>
	main()
File "./tools/train.py", line 178 in main
	train_detector()
File "mmdet/apis/train.py", line 177 in train_detector
	runner.run(data_loaders, cfg.workflow)
File "/home/zuo/code/mmcv/mmcv/runner/epoch_based_runner.py", line 177 in run
	epoch_runner(data_loaders[i], **kwargs)
File "/home/zuo/code/mmcv/mmcv/runner/epoch_based_runner.py", line 50 in train
	self.run_iter(data_batch, train_mode=True, **kwargs)
File "/home/zuo/code/mmcv/mmcv/runner/epoch_based_runner.py", line 29 in run_iter
	self.model.train_step(data_batch, self.optimizer, **kwargs)
File "/home/zuo/code/mmcv/mmcv/parallel/distributed.py", line 177 in train_step
	self.module.train_step(*inputs[0], **kwargs[0])
File "mmdet/models/detectors/base.py", line 238 in train_step
	losses = self(**data)
File "lib/python3.7/site-packages/torch/nn/modules/module.py", line 727  in _call_impl
	result = self.forward(*input, **kwargs)
File "/home/zuo/code/mmcv/mmcv/runner/fp16_utils.py", line 98 in new_func
	return old_func(*args, **kwargs)
File "mmdet/models/detectors/base.py", line 172  in forward
	return self.forward_train(img, img_metas, **kwargs)
File "mmdet/models/detectors/single_stage.py", line 82~83 in forward_train
	x = self.extract_feat(img), line 82  in forward_train
		extract_feat()                   "mmdet/models/detectors/single_stage.py:41", extract_feat(), 提取特征
	losses = self.bbox_head.forward_train(x, img_metas, gt_bboxes, gt_labels, gt_bboxes_ignore)	
		outs = self(x)                   "mmdet/models/dense_heads/yolox_head.py:196", forward(), 计算输出
		losses = self.loss(*loss_inputs) "mmdet/models/dense_heads/yolox_head.py:324", loss(), 计算loss
```



# Test

Traceback

```shell
_do_evaluate                             "mmdet/core/evaluation/eval_hooks.py:12"
  results = single_gpu_test(runner.model, self.dataloader, show=False)
    result = model(return_loss=False, rescale=True, **data)  "mmdet/apis/test.py:28"
      forward_test(self, imgs, img_metas, **kwargs)  "mmdet/models/detectors/base.py:112~154"
        simple_test()  "mmdet/models/detectors/single_stage.py"
          feat = self.extract_feat(img)  "mmdet/models/detectors/single_stage.py:41~46"
          self.bbox_head.simple_test(feat,..)
            simple_test() "mmdet/models/dense_heads/base_dense_head.py:334"
              simple_test_bboxes() "mmdet/models/dense_heads/dense_test_mixins.py:17~39"
                outs = self.forward(feats)
                  multi_apply() "mmdet/models/dense_heads/yolox_head.py:207"
        		results_list = self.get_bboxes()
        		  get_bboxes()  "mmdet/models/dense_heads/yolox_head.py:214~295"
          bbox2result(det_bboxes,..)     "mmdet/core/bbox/transforms.py:100~117"
  key_score = self.evaluate(runner, results)
    self.dataloader.dataset.evaluate()   "/home/zuo/code/mmcv/mmcv/runner/hooks/evaluation.py:359"
      evaluate()                         "mmdet/datasets/voc.py:28~105"
      	eval_map()                       "mmdet/core/evaluation/mean_ap.py:297~441"
```



# YOLOX迁移遇到的问题

主要是Dynamic Shape类问题

## dataset

由于数据集每张图片中`object`数量不固定，所以所模型输入tensor `gt_bboxes`和`gt_labels`的`shape`是不固定的。这就涉及到`dynamic shape`输入。彼时`MindSpore`对`dynamic shape`的支持还不完善，因此通过设置一个合理的最大`objects`数，将`gt_bboxes`和`gt_labels` padding到最大shape来规避`dynamic shape`问题。

另外增加有效object的标识作为输入tensor来表示哪些索引位置的object是真实的，哪些索引处的是padding的，以方便将padding对后续计算loss的影响降到最低。



## loss

loss计算流程中`get_in_gt_and_in_center_info`函数里最后一个步骤：

```python
# both in boxes and centers. shape:[num_fg, num_gt]
is_in_boxes_and_centers = (
	is_in_gts[is_in_gts_or_centers, :] &
    is_in_cts[is_in_gts_or_centers, :]
	)
```



`is_in_boxes_and_centers`的shape会随`is_in_gts_or_centers`的值不同而不同，也就产生了`dynamic shape`问题，为了规避这个问题，做了如下修改：

```python
is_in_boxes_and_centers = in_gts * in_cts
zeros = mnp.full_like(is_in_boxes_and_centers, 0)
is_in_boxes_and_centers = P.Select()(is_in_gts_or_centers_repeat, is_in_gts_and_centers, zeros)
```

通过Select算子，将`is_in_gts_or_centers_repeat`中`True`对应的元素赋值给`is_in_gts_and_centers`，`is_in_gts_or_centers_repeat`中`False`对应的位置padding成非法值（这里就是0）。

上面的例子是loss计算流程中`dynamic shape`的引入点，后续流程还有涉及到`dynamic shape`的地方，均按照同样的思想进行修改，同样要保证不影响loss的计算结果。



