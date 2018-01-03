---
title: 当nz-checkbox-group多选框组遇上必选校验
tags: Angular2,ng-zorro-antd,checkbox
grammar_cjkRuby: true
---
今天表单中用到ng-zorro-antd组件的多选框nz-checkbox-group，最开始用的是响应式表单的验证+响应式表单的验证，结果总是无法达到预期效果。代码如下：
Html
```
<div nz-form-item nz-row>
        <div nz-form-label nz-col [nzSpan]="4">
          <label for="one" nz-form-item-required>复选测试</label>
        </div>
         <div nz-form-control nz-col [nzSpan]="10" [nzValidateStatus]="getFormControl('one')">
          <nz-checkbox-group formControlName="one"  [(ngModel)]="oneOption" ></nz-checkbox-group>
          <div nz-form-explain *ngIf="getFormControl('one').dirty&&getFormControl('one').hasError('required')">通知范围必选</div>
        </div>
      </div>
```

