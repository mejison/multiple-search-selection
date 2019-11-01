<template>
  <div class="multiple-search-cover" :class="{invalid: hasError, opened}">
    <select name="options" multiple="multiple" class="d-none"></select>
    <span class="label">{{ label }}<span class="required" v-if="required">*</span></span>
    <span class="dropdown icon down" @click="onToggleDropdown" v-if="selected.length != 3">
      <svg class="down" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"/></svg>
      <svg class="up" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="18 15 12 9 6 15"/></svg>
    </span>
    <div 
      class="selected"
      v-for="(select, index) in selected"
      :key="`${index}-selected`"
    >
      {{ select.title }}
      <a href="#" class="cancel" @click.stop.prevent="onCancelOptionHandler(index)">&times;</a>
    </div>
    <input type="text" v-if="selected.length != 3" autocomplete="off" class="search" v-model="querySearch" @input="onSearchHandler" :placeholder="activePlaceholder" />
    <div class="menu">
      <div 
        class="item"        
        v-for="(option, index) in filteredOptions"
        :key="`${index}-option`"
        @click.stop="onSelectOption(option)"
        :class="{'active': isSelected(option), 'sub': currentOption && ! isSelected(option)}"
      >
        {{ option.title }}
        <span class="checkbox">
          <svg aria-hidden="true" focusable="false" data-prefix="fas" data-icon="check" class="svg-inline--fa fa-check fa-w-16 checked" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M173.898 439.404l-166.4-166.4c-9.997-9.997-9.997-26.206 0-36.204l36.203-36.204c9.997-9.998 26.207-9.998 36.204 0L192 312.69 432.095 72.596c9.997-9.997 26.207-9.997 36.204 0l36.203 36.204c9.997 9.997 9.997 26.206 0 36.204l-294.4 294.401c-9.998 9.997-26.207 9.997-36.204-.001z"></path></svg>
        </span>
      </div>
      <div class="item clear" v-if=" ! filteredOptions.length">
        Oops, nothing found
        <a href="#" class="btn-clear" @click.prevent="onClearEntryHandler">clear entry</a>
      </div>
    </div>
    <span v-if="inActivePlaceholder" class="description" :class="{'text-danger': hasError}">{{ inActivePlaceholder }}</span>
  </div>
</template>

<script>
  export default {
    name: 'MultipleSearch',
    props: {
      label: {
        type: String,
        default: 'Multiple Search',
      },
      placeholder: {
        type: String,
        default: 'Type to serach or choose from menu',
      },
      invalidmessage: {
        type: String,
        default: 'Please, select a valid account category',
      },     
      required: {
        type: Boolean,
        default: false,
      },
      options: {
        type: [Object, Array],
      },
      description: {
        type: String,
      },
      selectedoption: {
        type: Array,      
      },
    },
    data() {
      return {
        inActivePlaceholder: this.description,
        activePlaceholder: this.placeholder,
        invalid: false,
        opened: false,
        selected: [],
        querySearch: "",
        currentOption: false,
      }
    },
    mounted() {
      document.body.addEventListener('click', (e) => this.onClickBodyHandler(e), false)
    },
    computed: {
      filteredOptions() {
        let result = []
        let level = 0
        if (this.opened && ! this.selected.length) {
          result = this.options.flat(level)
        }

        if (this.currentOption && this.currentOption.children && this.currentOption.children.length) {
          result = [this.currentOption, ...this.currentOption.children]
        }

        if (this.querySearch) {
          let optionsFromAllLevels = this.options.reduce((ac, curr) => {
            let children = curr.children.map(c => {
              c.parent_id = curr.id
              return c
            })
            return [...ac, ...children]
          }, this.options)
          result = optionsFromAllLevels.filter(option => option.title.toLowerCase().indexOf(this.querySearch.toLowerCase()) + 1)
        }

        return result
      },
      hasError() {
        return this.invalid && this.querySearch && ! this.filteredOptions.length
      },
    },
    watch: {
      selected() {
        this.$emit('change', this.selected)
      },          
    },
    methods: {
      isSelected(option) {
        return this.selected.find(s => s.id === option.id)
      },
      onClickBodyHandler(e) {
        if ( ! e.target.closest('.multiple-search-cover')) {
          this.opened = false
          this.invalid = true
          if (this.querySearch) {
            this.inActivePlaceholder = this.invalidmessage
          }          
        }
      },
      onToggleDropdown() {
        this.opened = ! this.opened        
      },
      onSearchHandler() {
        this.invalid = false
        this.opened = true
        this.inActivePlaceholder = this.description
      },
      onClearEntryHandler() {
        this.inActivePlaceholder = this.description
        this.querySearch = ""
      },
      selectFromSearched(option) {
        let fined = this.findParents(option)
        this.selected = [...fined, option]
      },
      findParents(option) {
          if (option.parent_id) {
            let parents = []
            let check = true
            let optionsFromAllLevels = this.options.reduce((ac, curr) => {
            let children = curr.children.map(c => {
              c.parent_id = curr.id
              return c
            })
            return [...ac, ...children]
          }, this.options)
          
          while(check) {
            let parent = optionsFromAllLevels.find(o => o.id == option.parent_id)
            parents = [...parents, parent]
            if ( ! parent.parent_id) {
              break;
            }
          }
          return parents
        }
        
        return [option]
      },
      onSelectOption(option) {
        if (this.querySearch) {
          this.selectFromSearched(option)         
        } else {
          this.selected = [...this.selected, option]
        }
        
        this.querySearch = ""        
        this.activePlaceholder = ""
        this.currentOption = option
        if ( ! option.children || ! option.children.length) {
          this.opened = false
        }
      },
      onCancelOptionHandler(iteration) {
        this.selected = this.selected.filter((option, index) => {
          return index != iteration
        })
        
        if (this.selected.length) {
          this.currentOption = this.selected[this.selected.length - 1]
          this.opened = true
        } else {
          this.currentOption = false
        }

        if ( ! this.selected.length) {
          this.activePlaceholder = this.placeholder
        }
      },
    },
  }
</script>

<style lang="scss" scoped>
  
  @import url('https://fonts.googleapis.com/css?family=Proxima+Nova');

  .multiple-search-cover {
    background: #FFFFFF;
    border: 1px solid #C0CDD3;
    box-sizing: border-box;
    border-radius: 6px;
    padding: 25px 15px 10px 15px;
    position: relative;
    z-index: 0;
    font-family: Proxima Nova;    
    display: flex;
    flex-wrap: wrap;
    align-items: center;

    a {
      text-decoration: none;
    }

    .d-none {
      display: none;
    }

    .label {
      position: absolute;
      z-index: 1;
      top: 8px;
      left: 15px;
      font-size: 12px;
      line-height: 17px;

      .required {
        color: #EF3F61;
        font-size: 13px;
      }
    }

    .selected {
      display: inline-block;
      background: #5C94FE;
      color: #fff;
      font-size: 12px;
      color: #FFFFFF;
      text-transform: uppercase;
      border-radius: 10px;
      padding: 2px 10px;
      margin-right: 8px;

      .cancel {
        color: #fff;
        vertical-align: middle;
      }
    }

    .search {
      border: none;
      outline: none;
      width: auto;
      font-size: 16px;
      line-height: 24px;
      display: inline-block;
      min-width: 100px;
      align-items: center;
      color: #526C79;
      padding-right: 30px;
      flex-grow: 1;
    }

    .dropdown {
      position: absolute;
      top: 50%;
      right: 15px;
      transform: translateY(-50%);
      cursor: pointer;
      line-height: 0;
      user-select: none;

      .down, .up {
        display: none;
        stroke: #526C79;
        width: 17px;        
      }

      &.down {
        .down {
          display: inline-block;
        }
      }
    }

    .description {
      position: absolute;
      bottom: -20px;
      left: 15px;
      color: #526C79;
      font-size: 12px;

      &.text-danger {
       color: #EF3F61;
      }
    }

    .menu {
      position: absolute;
      z-index: 2;
      top: 61px;
      left: -1px;
      right: -1px;
      display: none;
      background: #fff;
      border: 1px solid #C0CDD3;
      border-bottom-left-radius: 6px;
      border-bottom-right-radius: 6px;      

      .item {
        padding: 15px;
        font-size: 16px;
        border-bottom: 1px solid #C0CDD3;
        color: #2A363D;
        cursor: pointer;
        display: flex;
        justify-content: space-between;
        align-items: center;        
        
        &:last-child {
          border-bottom: none;
        }
        
        &.active {
          .checkbox {
            border: 1px solid #4ED687;
            text-align: center;
            line-height: 16px;

            .checked {
              display: inline;              
              fill: #4ED687;
              width: 10px;              
            }
          }
        }

        &.sub {
          padding-left: 35px;
        }

        .checkbox {
          width: 16px;
          height: 16px;
          border: 1px solid #C0CDD3;
          border-radius: 50%;          

          .checked {
            display: none;
          }
        }

        .btn-clear {
          color: #5C94FE;
          font-size: 16px;          
        }
      }

      &.hidden {

      }
    }

    &.opened {
      border-bottom-left-radius: 0;
      border-bottom-right-radius: 0;  
      
      .dropdown {
        &.down {
          .up {
            display: inline-block;
          }

          .down {
            display: none;
          }
        }
      }

      .menu {
        display: block;
      }
    }

    &.invalid {
      border: 1px solid #EF3F61;
    }
  }
</style>