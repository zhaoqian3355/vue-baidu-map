<template>
  <span>
    <slot>
      <input v-if="!hasCustomerInput">
    </slot>
  </span>
</template>

<script>
import commonMixin from "../base/mixins/common.js";
import bindEvents from "../base/bindEvent.js";

export default {
  name: "pcstore-autocomplete",
  mixins: [commonMixin()],
  props: {
    types: {
      type: String
    },
    location: {
      type: String
    },
    sugStyle: {
      type: Object,
      default() {
        return {};
      }
    },
    cusInputId: {
      type: String
    },
    dragging: {
      type: Boolean,
      default: false
    },
  },
  watch: {
    types() {
      this.reload();
    },
    location() {
      this.reload();
    }
  },
  methods: {
    load() {
      const { BMap, map, $el, types, location, sugStyle,cusInputId,dragging} = this;
      var input = $el.querySelector("input");
      const customerInput = document.getElementById(cusInputId);
      if (customerInput) {
        this.hasCustomerInput = true;
        input = customerInput;
      }
      if (!input) {
        return;
      }
      this.originInstance = new BMap.Autocomplete({
        input,
        types,
        location: location || map,
        onSearchComplete: e => {
          const $sugs = document.querySelectorAll(".tangram-suggestion-main");
          for (const $sug of $sugs) {
            for (const name in sugStyle) {
              $sug.style[name] = sugStyle[name].toString();
            }
          }
          this.$emit("searchcomplete", e);
        }
      });

      // Support v-model
      this.originInstance.addEventListener("onconfirm", e => {
        const val = e.item.value;
        const address =
          val.province + val.city + val.district + val.street + val.business;
        this.$emit("input", address);
        map.clearOverlays(); //清除地图上所有覆盖物
        function myFun() {
          var pp = local.getResults().getPoi(0).point; //获取第一个智能搜索的结果
          map.centerAndZoom(pp, 15);
          var marker=new BMap.Marker(pp);
          if(dragging){
              marker.enableDragging()
          }else{
              marker.disableDragging()
          }
          map.addOverlay(marker); //添加标注
        }
        var local = new BMap.LocalSearch(map, {
          //智能搜索
          onSearchComplete: myFun
        });
        local.search(address);
      });

      bindEvents.call(this, this.originInstance);
    }
  },
  data() {
    return {
      hasCustomerInput: false
    };
  }
};
</script>
