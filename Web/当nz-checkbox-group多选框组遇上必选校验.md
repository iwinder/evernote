---
title: 当nz-checkbox-group多选框组遇上必选校验
tags: Angular2,ng-zorro-antd,checkbox
grammar_cjkRuby: true
---
今天表单中用到ng-zorro-antd组件的多选框nz-checkbox-group，最开始用的是响应式表单的验证+响应式表单的验证，结果总是无法达到预期效果。
## 初始代码及问题现象：
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
console.log(this.getFormControl('one').dirty); // 1
console.log(this.getFormControl('one').hasError('required')); // 2
console.log(this.validateForm.value.one); // 3
console.long(this.validateForm.invalid);// 4
```
结果发现
- 初始时：1、true，2、false，3、oneOption中的值，4、false
- 选择一个选项后：1、false，2、false，3、oneOption中的值+选中的value，4、false

从而始终无法触发显示 “通知范围必选”
## 第一次尝试
最开始尝试是将```this.validateForm.value.scopes```在提交时先赋值为[]，再检测checked状态，赋值。
```
this.validateForm.value.scopes = [];
for(const i in this.oneOption){
	if(this.oneOption[i].checked){
		this.validateForm.value.scopes.push(this.oneOption[i].value);
	}
}
```
这样写 ，```this.validateForm.value.one```是达到预期了，但提示效果依旧没出来。
继续探索，看到```getFormControl('one').hasError('required')```，既然有has，有没有set一类的？打了一下发现还真有一个this.getFormControl('one').setErrors()的方法，于是在上面的基础上想到这样一个方式：
```
if(this.validateForm.value.scopes.length == 0){
	this.getFormControl('one').setErrors(Validators.required);
}
```
发现一点用没有，搜索后找到另一种变形：
```
if(this.validateForm.value.scopes.length == 0){
	this.getFormControl('one').setErrors({'required':true});
}
```