---
<!-- layout: page -->
<!-- title: Covid Search -->
<!-- permalink: /covidsearch/ -->
---

<html class="no-js" lang="">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <title>COVID-19 Open Search</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="{{ site.baseurl }}/static/bootstrap.min.css">
        <link rel="stylesheet" href="{{ site.baseurl }}/static/AdminLTE.min.css">
            <link rel="stylesheet" href="{{ site.baseurl }}/static/font-awesome/4.2.0/css/font-awesome.min.css">
    <script src="{{ site.baseurl }}/static/bootstrap.min.js"></script>
    <script src="{{ site.baseurl }}/static/jquery-3.4.1.min.js"></script>
    <script src="{{ site.baseurl }}/static/app.min.js"></script>
    <style>
        #mydivheader {
            padding: 10px;
            cursor: move;
            z-index: 1032;
            background-color: #001f3e;
            color: #fff;
        }

        
        #mydiv {
            z-index: 1031;
        }
        .paper-content {
            max-height: 100px;
            overflow: hidden;
            text-overflow: ellipsis;
        }

    </style>
</head>

<body>

<header class="main-header">
    <nav class="navbar navbar-static-top bg-navy" style="margin-left:0;padding: 10px">
        <div class="main-container">
            <div class="navbar-header pull-left">
                <img src="{{ site.baseurl }}/static/pictures/icon.png"
                     width="30"
                     class="d-inline-block align-top" alt="" id="trigger-upload">
                Retrieval system for COVID-19 related papers based on <a href="https://pages.semanticscholar.org/coronavirus-research"> CORD19 dataset</a>
            </div>
        </div>
    </nav>
</header>


<div class="container">
    <div class="row" style="margin: 50px">
        <div class="col-md-12">
           <div class="input-group input-group">
                <input type="text" class="form-control" id="search-input">
                <span class="input-group-btn">
                      <button type="button" class="btn btn-info btn-flat" id="search-btn">Search</button>
                    </span>
            </div>
            <div class="form-group" style="margin-top: 10px">
                <label>Models</label><a href="#"><i class="fa fa-fw fa-question-circle"></i></a>
                <select class="form-control" id="model-select">
                    <option>TF-IDF</option>
                    <option>Count</option>
                    <option>BM25</option>
                    <option>FastText Embedding</option>
                    <option>BM25+FastText</option>
                </select>
            </div>
            <div class="form-group" style="margin-top: 30px">
                <label>Insights (Based on the tasks on <a
                        href="https://www.kaggle.com/allen-institute-for-ai/CORD-19-research-challenge/tasks">kaggle</a>)</label>
                <select class="form-control" id="kaggle-tasks">
                    <option>Select</option>
                </select>
            </div>
        </div>
    </div>
    <div class="row" style="margin: 10px">
          <div class="col-md-12">
            <div class="box box-solid">
                <div class="box-header with-border">
                    <h3 class="box-title" id="results-title">Search Results</h3><span id="results-count"></span>
                </div>
                    <div class="box-body" id="display-box">
                </div>
            </div>
        </div>
    </div>
</div>

<!-- <button id="ajax-test">click</button>
<div id="results"></div>
 -->

<div style="margin-bottom: 100px">
    &nbsp;
</div>

<footer class="main-footer"
        style="margin-left: 0;width:100%;height:50px;text-align: center;position: fixed;bottom: 0">
    <strong>Source code can be found in the <a href="#">Github Repository</a> @Congcong Wang</strong>
</footer>

<script type="text/javascript">

// $(document).ready(function(){
//  	alert("hello, everybody!");
// });
    var url="http://78.141.196.138:7000";


    $(function () {
        var kaggle_tasks = {
            "task1": "What is known about transmission, incubation, and environmental stability?",
            "task2": "What do we know about COVID-19 risk factors?",
            "task3": "What do we know about virus genetics, origin, and evolution?",
            "task4": "What do we know about vaccines and therapeutics?",
            "task5": "What do we know about non-pharmaceutical interventions?",
            "task6": "What has been published about medical care?",
            "task7": "Sample task with sample submission",
            "task8": "What do we know about diagnostics and surveillance?",
            "task9": "What has been published about information sharing and inter-sectoral collaboration?",
            "task10": "What has been published about ethical and social science considerations?"
        };

        $.each(kaggle_tasks, function (name, query) {
            var html = "<option>" + name + ": " + query + "</option>";
            $("#kaggle-tasks").append(html);
        });

        $("#kaggle-tasks").change(function () {
            var kaggle_task = $(this).val();
            if (kaggle_task != "Select") {
                var task_name = kaggle_task.split(":")[0];
                $.ajax({
                    type: "POST",
                    url: url+"/kaggle_task",
                    data: JSON.stringify({"task_name": task_name}),
                    contentType: "application/json",
                    success: function (data) {
                        var data_obj = JSON.parse(data);
                        var results = data_obj.result;
                        var len = data_obj.result.length;
                        $("#results-title").text(data_obj.response);
                        $("#results-count").text(" (" + len + ")");
                        $("#display-box").empty();
                        $.each(results, function (index, instance) {
                            var display_content = '<blockquote>' +
                                '<p class="fa-2x"><a href="' + instance.doi + '">' + instance.title + '</a></p>' +
                                '<div class="paper-insights"><span style="background-color: #ccffff;"> ' + instance.sentence + '</span></div>' +
                                '<p style="margin-top: 10px"><small><span>' + instance.authors.slice(0, 15) + ' et al. </span> Published: ' + instance.publish_time + '</small></p></blockquote>';
                            $("#display-box").append(display_content);
                        });
                    }
                });
            }
        });

        $('#search-input').keyup(function (e) {
            if (e.keyCode == 13) {
                var query = $(this).val();
                var model_type = $("#model-select").val();
                retrieve(query, model_type);
            }
        });

        $("#search-btn").click(function () {
            var query = $("#search-input").val();
            var model_type = $("#model-select").val();
            retrieve(query, model_type);
        });

        function retrieve(query, model_type) {
            if (query == "") {
                alert("enter a query !");
                return false;
            }
            $.ajax({
                type: "POST",
                url: url+"/search",
                data: JSON.stringify({"query": query, "model_type": model_type}),
                contentType: "application/json",
                success: function (data) {
                    var data_obj = JSON.parse(data);
                    var results = data_obj.result;
                    var len = data_obj.result.length;
                    $("#results-title").text(data_obj.response);
                    $("#results-count").text(" (" + len + ")");
                    $("#display-box").empty();
                    $.each(results, function (index, instance) {
                        var display_content = '<blockquote>' +
                            '<p class="fa-2x"><a href="' + instance.doi + '">' + instance.title + '</a></p>' +
                            '<div class="paper-content"> ' + instance.abstract + '</div>' +
                            '<p style="margin-top: 10px"><small><span>' + instance.authors.slice(0, 15) + ' et al. </span> Published: ' + instance.publish_time + '</small></p></blockquote>';
                        $("#display-box").append(display_content);
                    });
                }
            });
        }

    });



  $(function () {
            $('#ajax-test').bind('click', function () {
                $.ajax({
					corssDomain:true,
					type:"POST",
				  data: {"a":"das","b":"dsad"},
				  dataType: 'json',
				  xhrFields: {withCredentials:true},
				  url: "http://192.168.0.73:5000/json",
				  contentType: 'application/json; charset=utf-8',
				 success: function(jsondata){
				 	console.log(jsondata)
				 	$("#results").text(jsondata.result)
				   },
				     error: function(e) {
				     	console.log(e)
				        alert('Error: '+e);
				    } 
				});

            });
        });

// var commentdata="hello"
// var fbURL="http://192.168.0.73:5000/json?a=a-info&b=b-info";
// $.ajax({
//     url: "http://192.168.0.73:5000/json?a=a-info&b=b-info",
//     dataType: 'jsonp',
//     success: function (resp) {
//         alert(resp);
//         $("#results").text("return results:" + resp.result);
//     },
//     error: function(e) {
//         alert('Error: '+e);
//     } 
// });



// $.ajax({
//                 type: 'GET',
//                 data: "text=" + $(this).val(),
//                 url: "http://192.168.0.73:5000/json?a=a-info&b=b-info",
//                 success: function(data) {
//                     console.log(data);
//                 },
//                 error: function(jqXHR, textStatus, errorThrown) {
//                     console.log(jqXHR, textStatus, errorThrown);
//                     request.abort();
//                 }
//             });
// $.getJSON( "http://192.168.0.73:5000/json?a=a-info&b=b-info&callback=?", function( data ) {
//   alert(data)
// });


// $.ajax({
// url: "http://192.168.0.73:5000/json?a=a-info&b=b-info",
// dataType: 'jsonp',
// success: function(data){
// alert("data.result");
// }
// });



</script>

</body>
</html>


