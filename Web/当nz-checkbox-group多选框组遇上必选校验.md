---
title: 当nz-checkbox-group多选框组遇上必选校验
tags: Angular2,ng-zorro-antd,checkbox
grammar_cjkRuby: true
---
今天表单中用到ng-zorro-antd组件的多选框nz-checkbox-group，最开始用的是响应式表单的验证+响应式表单的验证，结果总是无法达到预期效果。代码如下：
问题.Html
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
问题.ts
这里仅列出关键代码部分
```
validateForm: FormGroup;
oneOption: any;
constructor(
    private fb: FormBuilder,
 ){}
ngOnInit() {
    this.oneOption = [
    { label: 'Apple', value: 'Apple', checked: true },
    { label: 'Pear', value: 'Pear', checked: false },
    { label: 'Orange', value: 'Orange', checked: false },
    ]
    this.validateForm = this.fb.group({
         one: [null, [Validators.required]],
    })
}
  getFormControl(name) {
    return this.validateForm.controls[name];
  }
```
为了找问题，在提交方法_submitForm（）中添加了几个打印
```
console.log(getFormControl('one').dirty); // 1
console.log(getFormControl('one').hasError('required')); // 2
console.log(this.validateForm.value.scopes); // 3
```
结果发现
初始时：1、true，2、false、3、oneOption中的值
