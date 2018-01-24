# h5-Slice-uploading
h5 切片上传
function uploadFile() {
            var shardCount = 0;
            var file = document.getElementById("uploadimg").files[0];
            var shardSize = 2 * 1024 * 1024;
            var successed = 0;
            var errortip = 0;
            if (file) {
                var fileSize = 0;
                if (file.size <= 5 * 1024 * 1024) {
                    fileSize = (Math.round(file.size * 100 / (1024 * 1024)) / 100).toString() + 'MB';
                    shardCount = Math.ceil(file.size / shardSize);  //总片数
                }
                else {
                    layer.open({
                        content: "上传图片不能大于5M",
                        skin: 'msg',
                        time: 2 //2秒后自动关闭
                    })
                    return false;
                }
                var uiid = new Date().getTime() + parseInt(Math.random() * 100);
                for (var i = 0; i < shardCount; ++i) {
                    //计算每一片的起始与结束位置
                    var start = i * shardSize,
                        end = Math.min(file.size, start + shardSize);
                    //构造一个表单，FormData是HTML5新增的
                    var formData = new FormData();
                    formData.append("data", file.slice(start, end));  //slice方法用于切出文件的一部分
                    formData.append("name", file.name);
                    formData.append("uiid", uiid);
                    formData.append("total", shardCount);  //总片数
                    formData.append("index", i + 1);        //当前是第几片
                    $.ajax({
                        url: '',  //分片传输
                        type: 'POST',
                        dataType: "json",
                        beforeSend: function () {
                            layer.open({
                                type: 2
                              , content: '图片上传中'
                            });
                        },
                        success: function (result) {
                            if (result.code == 0) {
                                successed++;
                                if (successed == shardCount) {
                                    $.ajax({
                                        url: '合并', type: 'post', dataType: "json", data: { ftype: file.type, name: file.name, uiid: uiid, total: shardCount }, success: function (rt) {
                                          
                                        }
                                    })
                                }
                            } else {
                                if (errortip == 0) {
                                    errortip++;
                                }
                            }
                        },
                        error: function (e) {
                           
                        },
                        data: formData,
                        cache: false,
                        contentType: false,
                        processData: false
                    });
                }
            }
        }
