[Rider](https://www.jetbrains.com/rider/) is a nice IDE for editing C# code.  It has a highlighting editor, debugger, and run configurations analogous to IntelliJ.

* Why isn't Rider code folding my gigantic files like Resolver.cs?

Dafny has large files.  E.g. Translator.cs is close to 20k lines.  Rider only runs analyses and folds code up to some file size less than that.  To allow Rider to work on such files, follow the advice on [this issue](https://rider-support.jetbrains.com/hc/en-us/community/posts/360000137704-No-analysis-has-been-performed-Document-size-has-exceeded-the-threshold-).  Namely, you need to increase the inspections threshold.