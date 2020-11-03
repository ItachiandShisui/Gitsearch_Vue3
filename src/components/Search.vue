<template>
  <div>
    <form>
      <input
        type="text"
        v-model="title"
        class="question"
        id="title"
        required
        autocomplete="off"
      />
      <label for="title"><span>Start typing!</span></label>
      <select v-model="currentFilter" name="filter" id="filter">
        <option v-for="item in filters" :id="item.id">{{ item.name }}</option>
      </select>
      <input type="submit" @click.prevent="handleSubmit" value="Search!" />
    </form>

    <p v-if="loading">Loading...</p>

    <div v-if="!loading && data && data.length" class="row">
      <div class="col">
        <div class="tbl-header">
          <table>
            <thead>
              <tr>
                <th>#</th>
                <th @click="handleNameFilter">Name</th>
                <th>Link</th>
                <th @click="handleStarFilter">Stars</th>
                <th @click="handleLicenseFilter">License</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(rep, index) in data" :key="rep.id">
                <td>{{ index + 1 }}</td>
                <td>{{ rep.name }}</td>
                <td><a v-bind:href="rep.svn_url" target="_blank">Github</a></td>
                <td>{{ rep.stargazers_count }}</td>
                <td>
                  <span v-if="rep.license"> {{ rep.license.name }}</span>
                  <span v-else> Not defined </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="tbl-content">
          <table></table>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { ref, onBeforeMount } from "vue";
export default {
  props: {},
  setup() {
    const title = ref("");
    const data = ref(null);
    const filters = ref(null);
    const currentFilter = ref("");
    const loading = ref(false);
    const initStarName = ref(true);
    const initStarSort = ref(true);
    const initLicenseSort = ref(true);
    const baseurl = "https://api.github.com";

    async function getFilters() {
      await fetch(`${baseurl}/licenses`).then((res) => {
        if (res.status === 200) {
          res.json().then((res) => {
            filters.value = res;
          });
        } else {
          console.log("ERROR");
        }
      });
    }

    async function handleSubmit() {
      let date = new Date();
      let dd = date.getDate();
      let mm = date.getMonth();
      let yyyy = date.getFullYear();
      if (dd < 10) {
        dd = "0" + dd;
      }
      if (mm < 10) {
        mm = "0" + mm;
      }
      date = `${yyyy}-${mm}-${dd}`;

      loading.value = true;
      if (currentFilter.value) {
        const filter = filters.value.find(
          (element) => element.name === currentFilter.value
        );
        await fetch(
          `${baseurl}/search/repositories?q=${title.value}+license:${filter.key}+created:${date}`
        )
          .then((res) => {
            if (res.status === 200) {
              res.json().then((res) => {
                data.value = res.items;
                loading.value = false;
              });
            } else {
              console.log("ERROR");
            }
          })
          .then(() => {
            loading.value = false;
          });
      } else {
        await fetch(
          `https://api.github.com/search/repositories?q=${title.value}+created:${date}`
        )
          .then((res) => {
            if (res.status === 200) {
              res.json().then((res) => {
                data.value = res.items;
                loading.value = false;
              });
            } else {
              console.log("ERROR");
            }
          })
          .then(() => {
            loading.value = false;
          });
      }
    }

    function handleNameFilter() {
      if (initStarName.value) {
        initStarName.value = false;
        data.value.sort((a, b) =>
          a.name > b.name ? 1 : b.name > a.name ? -1 : 0
        );
      } else {
        data.value = data.value.slice().reverse();
      }
    }

    function handleStarFilter() {
      if (initStarSort.value) {
        initStarSort.value = false;
        data.value.sort((more, less) => {
          return more.stargazers_count - less.stargazers_count;
        });
      } else {
        data.value = data.value.slice().reverse();
      }
    }

    function handleLicenseFilter() {
      if (initLicenseSort.value) {
        initLicenseSort.value = false;
        const noLicenseArray = data.value.filter(
          (element) => element.license === null
        );
        const filteredArray = data.value.filter(
          (element) => element.license !== null
        );
        filteredArray.sort((a, b) =>
          a.license.key > b.license.key
            ? 1
            : b.license.key > a.license.key
            ? -1
            : 0
        );
        data.value = [];
        data.value = filteredArray.concat(noLicenseArray);
      } else {
        data.value = data.value.slice().reverse();
      }
    }

    onBeforeMount(() => {
      getFilters();
    });

    return {
      loading,
      title,
      filters,
      currentFilter,
      data,
      handleSubmit,
      handleNameFilter,
      handleStarFilter,
      handleLicenseFilter,
    };
  },
};
</script>

<styles lang="css" scoped>
#filter {
  -webkit-appearance: none;
  -moz-appearance: none;
  -ms-appearance: none;
  appearance: none;
  outline: 0;
  box-shadow: none;
  border: 0 !important;
  background-image: none;
}
/* Remove IE arrow */
select::-ms-expand {
  display: none;
}
/* Custom Select */
#filter {
  position: relative;
  display: flex;
  width: 100%;
  max-width: 20em;
  height: 3em;
  line-height: 3;
  overflow: hidden;
  border-radius: 0.25em;
  background-color: #efefef;
}
#filter {
  margin-top: 10px;
  flex: 1;
  padding: 0 0.5em;
  color: black;
  cursor: pointer;
}
/* Arrow */
#filter::after {
  content: "\25BC";
  position: absolute;
  top: 0;
  right: 0;
  padding: 0 1em;
  background: #34495e;
  cursor: pointer;
  pointer-events: none;
  -webkit-transition: 0.25s all ease;
  -o-transition: 0.25s all ease;
  transition: 0.25s all ease;
}
/* Transition */
.select:hover::after {
  color: #f39c12;
}

#filter {
  font-family: "Ubuntu", sans-serif;
  display: flex;
}

/*INPUT*/

h1 {
  font-size: 28px;
  margin-bottom: 2.5%;
}

input,
span,
label,
textarea {
  font-family: "Ubuntu", sans-serif;
  display: block;
  margin: 10px;
  padding: 5px;
  border: none;
  font-size: 22px;
}
textarea:focus,
input:focus {
  outline: 0;
}

input.question,
textarea.question {
  font-size: 48px;
  font-weight: 300;
  border-radius: 2px;
  margin: 0;
  border: none;
  width: 80%;
  background: rgba(0, 0, 0, 0);
  transition: padding-top 0.2s ease, margin-top 0.2s ease;
  overflow-x: hidden; /* Hack to make "rows" attribute apply in Firefox. */
}
/* Underline and Placeholder */

input.question + label,
textarea.question + label {
  display: block;
  position: relative;
  white-space: nowrap;
  padding: 0;
  margin: 0;
  width: 10%;
  border-top: 1px solid red;
  -webkit-transition: width 0.4s ease;
  transition: width 0.4s ease;
  height: 0px;
}

input.question:focus + label,
textarea.question:focus + label {
  width: 80%;
}

input.question:focus,
input.question:valid {
  padding-top: 35px;
}

textarea.question:valid,
textarea.question:focus {
  margin-top: 35px;
}

input.question:focus + label > span,
input.question:valid + label > span {
  top: -100px;
  font-size: 22px;
  color: #333;
}

textarea.question:focus + label > span,
textarea.question:valid + label > span {
  top: -150px;
  font-size: 22px;
  color: #333;
}

input.question:valid + label,
textarea.question:valid + label {
  border-color: green;
}

input.question:invalid,
textarea.question:invalid {
  box-shadow: none;
}

input.question + label > span,
textarea.question + label > span {
  font-weight: 300;
  margin: 0;
  position: absolute;
  color: #8f8f8f;
  font-size: 48px;
  top: -66px;
  left: 0px;
  z-index: -1;
  -webkit-transition: top 0.2s ease, font-size 0.2s ease, color 0.2s ease;
  transition: top 0.2s ease, font-size 0.2s ease, color 0.2s ease;
}

input[type="submit"] {
  -webkit-transition: opacity 0.2s ease, background 0.2s ease;
  transition: opacity 0.2s ease, background 0.2s ease;
  display: block;
  opacity: 0;
  margin: 10px 0 0 0;
  padding: 10px;
  cursor: pointer;
  margin-bottom: 10px;
}

input[type="submit"]:hover {
  background: #eee;
}

input[type="submit"]:active {
  background: #999;
}

input.question:valid ~ input[type="submit"],
textarea.question:valid ~ input[type="submit"] {
  -webkit-animation: appear 1s forwards;
  animation: appear 1s forwards;
}

input.question:invalid ~ input[type="submit"],
textarea.question:invalid ~ input[type="submit"] {
  display: none;
}

@-webkit-keyframes appear {
  100% {
    opacity: 1;
  }
}

@keyframes appear {
  100% {
    opacity: 1;
  }
}

/*TABLE*/

table {
  width: 100%;
  border-collapse: collapse;
}
/* Zebra striping */
tr:nth-of-type(odd) {
  background: #eee;
}
th {
  background: #333;
  color: white;
  font-weight: bold;
}
td,
th {
  padding: 6px;
  border: 1px solid #ccc;
  text-align: left;
}
/*
Max width before this PARTICULAR table gets nasty
This query will take effect for any screen smaller than 760px
and also iPads specifically.
*/
@media only screen and (max-width: 760px),
  (min-device-width: 768px) and (max-device-width: 1024px) {
  /* Force table to not be like tables anymore */
  table,
  thead,
  tbody,
  th,
  td,
  tr {
    display: block;
  }

  /* Hide table headers (but not display: none;, for accessibility) */
  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  tr {
    border: 1px solid #ccc;
  }

  td {
    /* Behave  like a "row" */
    border: none;
    border-bottom: 1px solid #eee;
    position: relative;
    padding-left: 50%;
  }

  td:before {
    /* Now like a table header */
    position: absolute;
    /* Top/left values mimic padding */
    top: 6px;
    left: 6px;
    width: 45%;
    padding-right: 10px;
    white-space: nowrap;
  }

  /*
Label the data
*/
  td:nth-of-type(1):before {
    content: "#";
  }
  td:nth-of-type(2):before {
    content: "Name";
  }
  td:nth-of-type(3):before {
    content: "Link";
  }
  td:nth-of-type(4):before {
    content: "Stars";
  }
  td:nth-of-type(5):before {
    content: "License";
  }
}
</styles>
