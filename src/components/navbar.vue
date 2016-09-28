<template>
	<nav class="uk-navbar navbar">
		<div class="navbar-nav uk-container-center">
			<ul class="uk-navbar-nav">
				<li><a href="javascript:void(0)" @click="doNothing">{{ userName  }}</a></li>
				<li class="run-btn"><a href="javascript:void(0)" @click="runCode">執行 <i class="uk-icon-play"></i></a></li>
				<li><a v-link="{ path: '/' + $route.params.courseId + '/' + $route.params.qn, exact: true}">程式撰寫</a></li>
				<li><a v-link="{ path: '/' + $route.params.courseId + '/' + $route.params.qn + '/logs' }">執行記錄</a></li>
				<li><a href="javascript:void(0)" @click="reset">重設此題</a></li>
				<li class="uk-parent" data-uk-dropdown="{mode:'click'}">
					<a href="javascript:void(0)">Input <i class="uk-icon-caret-down"></i></a>
					<stdin></stdin>
				</li>
			</ul>
			<div class="uk-navbar-flip">
				<ul class="uk-navbar-nav">
					<li><a href="javascript:void(0)" @click="checkPrev"><i class="uk-icon-arrow-left"></i> 上一題</a>
					</li>
					<li><a v-link="{ path: '/' + $route.params.courseId + '/' + $route.params.qn + '/list' }">題目清單</a></li>
					<li><a href="javascript:void(0)" @click="checkNext">下一題 <i class="uk-icon-arrow-right"></i></a>
					</li>
				</ul>
			</div>
		</div>
	</nav>
</template>

<script>
/* global UIkit, BiwaScheme, puts */
import store from './../lib/store'
import Stdin from './stdin.vue'
import { checkAnswer } from './../lib/util.js'

const COMPILE_JAVA_SERVER = 'http://140.125.90.231:300';
const maxCompileJavaServer = 3;
var javaServerId = 0;

const COMPILE_C_SERVER = 'http://140.125.90.231:400';
const maxCompileCServer = 1;
var cServerId = 0;

const COMPILE_CPP_SERVER = 'http://140.125.90.231:500';
const maxCompileCPPServer = 1;
var cppServerId = 0;

const SUBMIT_SERVER = 'http://140.125.90.231:8888/submit'
//const SUBMIT_SERVER = 'http://localhost:8888/submit'

var nextEnabled = true;
var prevEnabled = true;
var runCodeEnabled = true;

export default {
        data() {
	       return {
	       	      userName: '' + store.getUserName() + '       '
	      }
	},

	methods: {
		// hack for display user name
		doNothing() {
		     return;
		     },
		runCode() {
		    if (runCodeEnabled) {
			var qn = this.$route.params.qn,
				userCode = store.getUserCode(qn),
//				userCode = store.getCode();
				stdin = store.getStdin(qn),
				title = store.getQuizData(qn).title,  // tile is file name of code
				lang = store.getLanguage();

			if (typeof userCode === 'undefined') return;

			var codeData = {
				code: userCode,
				stdin: stdin,
				language: lang,
				fileName: title
			};

			let data = {
				output: '',
				userCode: userCode,
				stdin: stdin,
				errors: ''
			};

			// C
			if (lang == 'C') {
                            runCodeEnabled = false;
                            if (cServerId === maxCompileCServer) {
                                cServerId = 0;
                            	this.$http.post(COMPILE_C_SERVER + cServerId + '/compile', codeData).then((response) => {
			           var outStr = response.data.output;
				   var errorStr = response.data.errors;
  				   data.output =   outStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   data.errors = errorStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   this.finish(data);})
			        cServerId++;
                            } else {
                                this.$http.post(COMPILE_C_SERVER + cServerId + '/compile', codeData).then((response) => {
			           var outStr = response.data.output;
				   var errorStr = response.data.errors;
  				   data.output =   outStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   data.errors = errorStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   this.finish(data);})
                            	cServerId++;
                            }
		            setTimeout(function () {
		       	        console.log('prev-enabled');
			        runCodeEnabled = true;
			    }, 10000);
			}

			// C++
			if (lang == 'CPP') {
                            runCodeEnabled = false;
                            if (cppServerId === maxCompileCPPServer) {
                                cppServerId = 0;
                            	this.$http.post(COMPILE_CPP_SERVER + cppServerId + '/compile', codeData).then((response) => {
			           var outStr = response.data.output;
				   var errorStr = response.data.errors;
  				   data.output =   outStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   data.errors = errorStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   this.finish(data);})
                                cppServerId++;
                            } else {
                                this.$http.post(COMPILE_CPP_SERVER + cppServerId + '/compile', codeData).then((response) => {
			           var outStr = response.data.output;
				   var errorStr = response.data.errors;
  				   data.output =   outStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   data.errors = errorStr.replace(/\/usr\/src\/app\/scripts\/\d\d\//g, '');
				   this.finish(data);})
                            	cppServerId++;
                            }
			    setTimeout(function () {
		       	        console.log('prev-enabled');
			        runCodeEnabled = true;
			    }, 10000);
			}

			// JavaScript
			if (lang == 'JavaScript') {
				data.output = this.runJavaScript(codeData.code);
				this.finish(data);
			}

			// Scheme
			if (lang == 'Scheme') {
				data.output = this.runScheme(codeData.code);
				this.finish(data);
			}

			if (lang == 'Java') {
                           runCodeEnabled = false;
			   if (javaServerId === maxCompileJavaServer) {
			       javaServerId = 0;
			       this.$http.post(COMPILE_JAVA_SERVER + javaServerId + '/compile', codeData).then((response) => {
			           var outStr = response.data.output;
				   var errorStr = response.data.errors;
  				   data.output =   outStr.replace(/\/usr\/src\/app\/scripts\/\d\d/g, '');
				   data.errors = errorStr.replace(/\/usr\/src\/app\/scripts\/\d\d/g, '');
				   this.finish(data);})
			       javaServerId++;
			   } else {
			       this.$http.post(COMPILE_JAVA_SERVER + javaServerId + '/compile', codeData).then((response) => {
			           var outStr = response.data.output;
				   var errorStr = response.data.errors;
  				   data.output =   outStr.replace(/\/usr\/src\/app\/scripts\/\d\d/g, '');
				   data.errors = errorStr.replace(/\/usr\/src\/app\/scripts\/\d\d/g, '');
				   this.finish(data);})
			      javaServerId++;
			   }
			   setTimeout(function () {
		       	       console.log('prev-enabled');
			       runCodeEnabled = true;
			   }, 10000);
		       }
//		       data.output = 'System error: unknown language: ' + lang;
//		       this.finish(data);
		       return;
		   } else {
		       alert('Please wait!');
		       return;
		   }
		        
		},

		runPython(code, data) {

		},

		runJavaScript(code) {
			let buffer = [];
			let stdout = '';
			console.log = function () {
				buffer.push(arguments);
			}

			{
				try {
					eval(code);
					buffer.forEach((e) => {
						Array.prototype.slice.call(e).forEach((str) => (stdout += (str.toString() + '\n')));
					});
				} catch (err) {
					stdout = err.message;
				}
			}
			return stdout;
		},

		runScheme(code) {
			var stdout = '';
			var scheme = new BiwaScheme.Interpreter((e) => { stdout = e.message });

			scheme.evaluate(code, (result) => {
				if (result !== undefined && result !== BiwaScheme.undef) {
					for (var i in result) {
						stdout = stdout + result[i] + '\n';
                                    	}
				}
			});

			return stdout;
		},

		finish(data) {
			var quizData = store.getQuizData(this.$route.params.qn);
			var tmpStdout = quizData.stdout;
			var tmpStdout2 = tmpStdout.replace(/請按任意鍵繼續 . . ./, '');
			var result = checkAnswer(data.output.trim(), tmpStdout2.trim());
			var loggedIn = store.isLoggedIn();
			store.addLog(quizData.title, data.userCode, data.stdin, data.output + data.errors, result);
			
			if (!loggedIn) {
				UIkit.notify('警告：未正確登入，練習結果將不會儲存。', {status: 'warning'});
			} else {
				this.submit(data.userCode, result);
			}

			this.$route.router.go('/' + this.$route.params.courseId + '/' + this.$route.params.qn + '/logs');
		},

		submit(code, result) {
			var classId = store.getClassId(),
				courseId = this.$route.params.courseId.split('-')[0],
				lessonId = this.$route.params.courseId.split('-')[1],
				qn = this.$route.params.qn,
				qid = store.getQuizData(qn).qid;

			var type;

			if (result) {
				type = 'test_ok';
			} else {
				type = 'test_error';
			}

			var submitData = {
				code,
				uid: store.getUser(),
				type
			};

			try {
				this.$http.post(`${SUBMIT_SERVER}/${classId}/${courseId}/${lessonId}/${qid}`, submitData).then((response) => {
					if (response.data === 'ok') {
						UIkit.notify('練習結果已成功儲存！', {status: 'success'});
					} else {
						UIkit.notify('練習結果儲存失敗！', {status: 'danger'})
					}
				});
			} catch (e) {UIkit.notify('與伺服器連線失敗，練習結果無法儲存！', {status: 'danger'})}
		},

		reset() {
			UIkit.modal.confirm('確定要將此題清除重寫？', () => {
				store.resetCode(this.$route.params.qn);
			});
		},

		checkPrev() {
		 	if (prevEnabled) {
				if (parseInt(this.$route.params.qn) === 1) {
					alert('這已經是第一題了！');
					this.$route.router.go('/' + this.$route.params.courseId + '/1');
					return false;
				}
				let courseId = this.$route.params.courseId,
				    qn = parseInt(this.$route.params.qn) - 1;
				// to prevent the user from pushing the next button to fast, which can cause problems.
				prevEnabled = false;
				setTimeout(function () {
					console.log('prev-enabled');
					prevEnabled = true;
					}, 1000);
			        this.$route.router.go(`/${courseId}/${qn}`);
			}
		},

		checkNext() {
			if (nextEnabled) {
				if (parseInt(this.$route.params.qn) === store.getQuizCount()) {
				alert('這已經是最後一題了！');
				this.$route.router.go('/' + this.$route.params.courseId + '/' + store.getQuizCount());
				return false;
				}
				let courseId = this.$route.params.courseId,
				    qn = parseInt(this.$route.params.qn) + 1;
				// to prevent the user from pushing the next button to fast, which can cause problems.
			        nextEnabled = false;
				setTimeout(function () {
					console.log('next-enabled');
					nextEnabled = true;
					}, 1000);
				this.$route.router.go(`/${courseId}/${qn}`);
			} 
		},

		setLang(index) {
			this.currentLang = index
		}
	},

	components: {
		Stdin
	}
}
</script>

<style lang="stylus">
@require './../styles/vars'
@require './../styles/functions'

.navbar
	background main-color
	color #FFF
	height navbar-height
	box-shadow  0 2px 2px 0 rgba(0,0,0,.14),0 3px 1px -2px rgba(0,0,0,.2),0 1px 7px 0 rgba(0,0,0,.12)
	position relative
	z-index 300
	.run-btn
		margin-right 22px
	li a
		color #FFF
		font-size 16px
		height navbar-height
		line-height navbar-height
		@media screen and (max-width: 1100px)
			font-size 14px
		&:hover, &:focus, &:active
			background whiteA(.10)
			color #FFF
		i
			font-size 14px
			@media screen and (max-width: 1100px)
				font-size 12px
	li a.v-link-active
		background whiteA(.20)
	li.uk-open a
		&:hover, &:focus, &:active
			color #444
			background #f5f5f5
			
	.language-dropdown
		box-shadow 0 2px 2px 0 rgba(0,0,0,.16),0 3px 1px -2px rgba(0,0,0,.2),0 2px 6px 0 rgba(0,0,0,.12),0 0 3px rgba(0,0,0,.4)
		padding 5px 15px
		a
			color #444
			line-height 20px
			height @line-height
			&:hover, &:focus, &:active
				background #E5E5E5!important

.navbar-nav
	width navbar-width
	@media screen and (max-width: 1100px)
		width 100vw

.uk-notify-message
	box-shadow 1px 2px 16px #888
</style>
