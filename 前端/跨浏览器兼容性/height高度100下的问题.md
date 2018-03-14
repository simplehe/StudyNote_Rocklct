## height100%下浏览器之间的问题
 - 在Firefox、Chrome中,DOCTYPE为XHTML 1.0 Strict下默认html和body的高度是根据内容定，在body下设置一个高度为height:100%的div并不会充满整个浏览器窗口，必须同时设置html和body的高度为100%方可使该div充满整个浏览器窗口

 - body的overflow默认值：ie6下overflow-y=scroll,overflow-x=auto;Firefox、Chrome默认均为auto.因此如果不设置body的overflow，默认状态下ie6就会显示**不合理的垂直滚动条**，设overflow=auto或hidden均可解决
 - 各种浏览器body的margin都不是0，因此要保证div100%高并且正好充满整个窗口，须将body的margin设为0

**总结:html和body的高度设为100%，body设margin=0、overflow=hidden，div设overflow=auto，各浏览器表现一至，效果最佳。**