<template>
  <div>
    <h3>Description</h3>
    <p>I made a form component that is designed to autocorrect on first blur. The way it should behave is if you're touching the form for the first time, and you input an invalid name as checked by Vuelidate, it will change the label to the inputted name and autocorrect the name.</p>
    <p>This works as expected if an invalid name is entered that includes new letters or numbers. <strong>However if only symbols are added, the name is not updated to the corrected name and I cannot figure out why.</strong></p>
    <p>e.g. if I have the name: name_63 and change it to name_63#9 it autocorrects and displays to name_639.</p>
    <p>However, if I change it to name_63# the display remains name_63# instead of changing to name_63.</p>
    <p>In a repl environment, the regexp autocorrect feature also works 100% of the time.</p>
    <p>At first I tried using the .lazy method of getting Vuelidation on blur which didn't work, so I moved to the second implementation described in the docs, which binds to :value and manually mutates it. As a part of that I accessed the Vuelidate model rather than the component's model directly (e.g. $v.newColumnName.$model vs this.newColumnName) but they appear to be interchangeable.</p>
    <p>It's also weird because in the data properties in both the component itself and the parent, the Vuelidate model, the events emitted, and any console.logs have the correct value. Once I blur again it reverts the value to what I can see on the screen.</p>
    <p>Binding to v-model instead produces the autocorrected name as expected but takes away the ability to Vuelidate on blur.</p>
    <p>However, switching el-input for an input tag displays the correct behavior. So what is different between el-input's v-model and value attributes, and between el-input and input's value attributes? And why does it only affect symbol input?</p>
    <p>The only difference between the two cases as far as I can tell is if you update with symbols only then it adds an empty string instead of any sanitized characters to the input value. However I'm not sure why this would be an issue.</p>
  </div>
</template>