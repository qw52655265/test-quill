<template>
  <div :id="`${id}-${randomId}`" class="quill-editor flex column stretch auto">
    <div
      :id="`quill-toolbar-${id}-${randomId}`"
      ref="toolbarRef"
      class="editor"
    >
      <button class="ql-bold" />
      <button class="ql-italic" />
      <button class="ql-blockquote" />
      <button class="ql-hr el-icon-minus Minus">
        <el-icon><Minus /></el-icon>
      </button>
      <select class="ql-align" />
      <select class="ql-color" />
      <button class="ql-list" value="ordered" />
      <button class="ql-list" value="bullet" />
      <select class="ql-header" />
    </div>
    <div
      :id="`quill-editor-${id}-${randomId}`"
      ref="editorRef"
      class="editor auto flex column stretch"
    />
  </div>
</template>

<script>
import "quill/dist/quill.snow.css";
import Quill from "quill";
import { onBeforeUnmount, onMounted, reactive, ref, toRefs, watch } from "vue";
import { useFormItem } from "element-plus";
Quill.register(Quill.import("attributors/style/align"), true);
class DividerBlot extends Quill.import("blots/block/embed") {}
DividerBlot.blotName = "divider";
DividerBlot.tagName = "hr";
Quill.register(DividerBlot);
export default {
  name: "QuillEditor",
  props: {
    modelValue: { type: String, default: "" },
    disabled: { type: Boolean, default: false },
  },
  setup(props, context) {
    const id = ref(new Date().getTime());
    const randomId = ref(Math.random().toString().replace("0.", ""));
    const editorRef = ref(null);
    const toolbarRef = ref(null);
    const { formItem } = useFormItem();
    const data = reactive({
      html: undefined,
    });
    let quill = reactive({});
    watch(
      () => props.modelValue,
      () => {
        if (props.modelValue !== data.html) {
          pasteHTML();
        }
      }
    );
    onMounted(() => {
      quill = new Quill(`#quill-editor-${id.value}-${randomId.value}`, {
        theme: "snow",
        modules: {
          clipboard: {
            matchVisual: false,
          },
          toolbar: {
            container: toolbarRef.value,
            handlers: {
              hr() {
                const range = quill.getSelection(true);
                quill.insertEmbed(range.index, "divider", 0);
                quill.setSelection(range.index + 2);
              },
            },
          },
        },
      });
      quill.on("text-change", () => {
        data.html = editorRef.value.children[0].innerHTML
          .replace(/<span class="ql-cursor">.*<\/span>/g, "")
          .replace(/<p><br><\/p>$/, "")
          .replace(/<p><img/g, '<p class="img-wrap"><img');
        context.emit("update:modelValue", data.html);
        formItem?.validate("change");
        formItem?.validate("blur");
      });
      pasteHTML();
      if (props.disabled) {
        quill.enable(false);
      }
    });
    onBeforeUnmount(() => {
      quill = {};
    });
    const pasteHTML = () => {
      if (props.modelValue) {
        if (props.modelValue.includes("</")) {
          quill.clipboard.dangerouslyPasteHTML(
            props.modelValue.replace(/<p class="img-wrap"><img/g, "<p><img")
          );
        } else {
          const paragraphs = props.modelValue.split("\n");
          let html = "";
          for (const paragraph of paragraphs) {
            html += `<p>${paragraph}</p>`;
          }
          quill.clipboard.dangerouslyPasteHTML(html);
        }
      } else {
        quill.setText("");
      }
    };
    return {
      id,
      randomId,
      editorRef,
      toolbarRef,
      ...toRefs(data),
      pasteHTML,
    };
  },
};
</script>
