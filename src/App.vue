<template>
  <div>
    <div class="editable-element" ref="editor" contenteditable v-html="text" @blur="onBlur"></div>
    <div class="comment" >Sometimes you need to click twice to remove the token</div>
    <button class="btn-add-token" v-on:click="onAddToken" >Add token</button>
  </div>
</template>

<script>

import { onMounted, ref } from 'vue'

export default {
  name: 'App',
  setup(){

    const editor = ref(null)
    const text = ref("test test test")
    const startPositionCaret = ref(null)
    const tokenNumber = ref(0)

    const createTokenHtml = (tokenId) => {
      return `<button contenteditable="false" class="token"> Token - ${tokenId} <span class="close-button" id="${tokenId}">X</span></button>`
    }
    // inspired by https://codepen.io/sakthivelsekar33/pen/JvVZXr
    const onBlur = (evt) => {
      text.value = evt.target.innerHTML
    }

    const onAddToken = () => {
      const stringTokenNumber = tokenNumber.value.toString()
      const token = createTokenHtml(stringTokenNumber)

      if(startPositionCaret.value){
        const firstPartText = text.value.substring(0, startPositionCaret.value)
        const secondPartText = text.value.substring(startPositionCaret.value)
        text.value = `${firstPartText}${token}${secondPartText}`

        // adding an event listener to the token element wasnt working
        // the event listener was somwhow getting removed when user made
        // focus again on the content editable element
        // setTimeout(()=>{
        //   document.getElementById(stringTokenNumber).addEventListener("click", () => onRemoveToken(stringTokenNumber), true);
        // })

      }else if(startPositionCaret.value === 0){
        text.value = `${token}${text.value}`
      }else{
        text.value = `${text.value}${token}`

        // setTimeout(()=>{
        //   document.getElementById(stringTokenNumber).addEventListener("click", () => onRemoveToken(stringTokenNumber), true);
        // },0)
        
      }
      tokenNumber.value = tokenNumber.value + 1
    }

    const onRemoveToken = (id) => {
      const token = createTokenHtml(id)
      const newTextEditableElement = text.value.replace(token, "")
      text.value = newTextEditableElement
    }

  // https://stackoverflow.com/questions/4767848/get-caret-cursor-position-in-contenteditable-area-containing-html-content
  // solution with 15 votes
  function getCaretPosition (node) {
    var range = window.getSelection().getRangeAt(0),
        preCaretRange = range.cloneRange(),
        caretPosition,
        tmp = document.createElement("div");

    preCaretRange.selectNodeContents(node);
    preCaretRange.setEnd(range.endContainer, range.endOffset);
    tmp.appendChild(preCaretRange.cloneContents());
    caretPosition = tmp.innerHTML.length;
    return caretPosition;
}


    function reportSelection() {
      startPositionCaret.value = getCaretPosition(editor.value)
    }

    const onClickEditableElement = (event) => {
      //https://javascript.info/event-delegation
      let target = event.target;
      if(target.tagName === "SPAN"){
        onRemoveToken(target.id.toString())
      }else{
        reportSelection()
      }
    }

    onMounted(()=>{
        editor.value.addEventListener("selectionchange", () => reportSelection(), false);
        editor.value.addEventListener("click", (event) => onClickEditableElement(event), false);
        editor.value.addEventListener("keyup", () => reportSelection(), false);
    })
    return {
      onAddToken,
      text,
      editor,
      onBlur,
      onRemoveToken
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.btn-add-token{
  margin-top: 20px;
}

.comment{
  margin-top: 10px;
}

.editable-element{
  border: 1px solid black;
}

.token {
  border: 1px solid red;
  padding: 4px;
  background-color: yellow;
}

.close-button{
  font-size: 20px;
  font-weight: 700;
  padding: 2px;
  cursor: pointer;
}

</style>
