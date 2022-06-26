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

