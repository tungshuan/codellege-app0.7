<template>
	<div class="editor-container">
		<div id="editor" class="editor"></div>
	</div>
</template>

<script>
import Firebase from 'firebase'
import store from './../lib/store'

var dontSetUserCode = false;

export default {
	data() {
		return {
		        ref: 'https://plweb-tutor.firebaseio.com/' + store.getClassId() + '-' + store.getCourseId() + '-' + store.getLessonId() + '-' + store.getUser() + '/codepad/',
			codeMirror: {},
			firepad: {},
			isInit: false
		}
	},
 // this.$route.params.qn 

	ready() {
		var languageMode = '';
		var language = store.getLanguage();
		if (language == 'Scheme') {
		   languageMode = "text/x-scheme";
		} else if (language == 'JavaScript') {
		   languageMode = "text/javascript";
		} else if (language == 'C') {
		   languageMode = "text/x-csrc";
		} else if (language == 'CPP') {
		   languageMode = "text/x-c++src";
		} else if (language == 'Python') {
		   languageMode = "text/x-python";
		} else {
		   languageMode = "text/x-java";
		} 
		console.log(this.ref);
		this.codeMirror = CodeMirror(document.getElementById('editor'), {
			mode: languageMode,
			matchBrackets: true,
			lineNumbers: true,
			indentUnit: 4,
			viewportMargin: Infinity,
			lineWrapping: true
		});

		this.codeMirror.on('changes',() => {
		    	if (dontSetUserCode) {
		       	        dontSetUserCode = false;
		       		return;
			}
			store.setUserCode(this.$route.params.qn, this.codeMirror.getValue());
		});
		store.setCourseId(this.$route.params.courseId);
		store.onResetCode((partCode) => {
			this.codeMirror.setValue(partCode);
		});

		this.loadQuiz();
		this.isInit = true;
	},

	route: {
		data(transition) {
			if (this.isInit === false) return;
			if (typeof this.firepad.dispose !== 'undefined') {
			     console.log('doing dispose');
   	   			this.firepad.dispose();
				this.firepad = {};
		        }
			dontSetUserCode = true;
			this.codeMirror.setValue('');
			this.loadQuiz();
		}
	},

	methods: {
		loadQuiz() {
			store.updateQ(this.$route.params.qn, (data) => {
				if (typeof data === 'undefined') {
					console.log('沒題目了！');
					return;
				}
				console.log('Qn: ' + this.$route.params.qn);
				this.firepad = Firepad.fromCodeMirror(new Firebase(this.ref + this.$route.params.qn), this.codeMirror);
				store.setFirepad(this.firepad);
				var qnum = this.$route.params.qn;
				var cm = this.codeMirror;
				var dp = data.part;
				var da = data.answer;
				this.firepad.on('ready', function () {
				    if (store.getCode() == '') {
//				        console.log('qnum: ' + qnum);
//				        var userCode = store.getUserCode(qnum);
//					console.log('userCode: ' + userCode); 
//				        if (typeof userCode !== 'undefined') {
//					    console.log('seting userCode');
//					    cm.setValue(userCode);
//				        }
				        if (typeof dp !== 'undefined') {
					    console.log('setting dp');
					    cm.setValue(dp);
				        } else {
					    console.log('setting da');
					    cm.setValue(da);
				        }
				    }
			       })
			})
		}
	}
}
</script>

<style lang="stylus">
@require './../styles/vars'

.editor-container
	display flex

.editor, .firepad, .CodeMirror
	flex 1
	height full-height
	pre
		font-family Roboto Mono, monospace

</style>
