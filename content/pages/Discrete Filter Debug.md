---
title: Discrete Filter Debug
tags:
categories:
date: 2022-11-23
lastMod: 2022-11-23
---

This is a test/examle post.

How to

```
>>> import torch
>>> import matplotlib.pyplot as plt

>>> def plot(pr, sm, po, channel):
...     for i, d in enumerate([pr[channel], sm[channel], po[channel], (sm*pr)[channel]]):
...             plt.subplot(1, 4, i+1)
...             plt.imshow(d)
...     plt.show()

>>> pr = torch.load('/export/hartzj/TakeLidOffSaucepan/demos_gt/encodings/dcm_10_keypoints_encoder-resnet101bs128split9010/new_ego/2022_05_03-14_34_44/cam_w_prior/80.dat')
>>> po = torch.load('/export/hartzj/TakeLidOffSaucepan/demos_gt/encodings/dcm_10_keypoints_encoder-resnet101bs128split9010/new_ego/2022_05_03-14_34_44/cam_w_post/80.dat')
>>> sm = torch.load('/export/hartzj/TakeLidOffSaucepan/demos_gt/encodings/dcm_10_keypoints_encoder-resnet101bs128split9010/new_ego/2022_05_03-14_34_44/cam_w_sm/80.dat')
```

Results for sigma=2

  + Channels: 10, 2, 14

  + While on 2 it resolves the multimodality, on channel 10 it does not manage to do it. On Channel 14 it further manages to sharpen the cluster, but sometimes it also fails to do that.

  + ~~I think the noise component in the motion model is too strong.~~

![df-no-gain.png](/assets/df-no-gain_1659603466739_0.png)

![df-not-quite-strong-enough.png](/assets/df-not-quite-strong-enough_1659603495255_0.png)

![df-fixes-clutster.png](/assets/df-fixes-clutster_1659603501793_0.png)

  + If you look at successive frames of the same channel, sometimes multi-modalities would spring up later in the trajectory.

  + Problem was that the softmax was stored as the prior for the next step, not the posterior.

  + Fixing it, resolves that and does in fact improve kp localization.

  + The numerical effect is not very large though.

    + Could be that collapse of multi-modalities is not correct, but stable across a trajectory. In that case should still see improvement in policy learning.

    + But, I also observed some keypoints still moving. Could be that this only happens when the kp is moving outside too much.

    + Could be that smaller sigma helps.
