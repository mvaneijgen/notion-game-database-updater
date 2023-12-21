<script>
import secrets from "../../public/secrets.json";

export default {
  data() {
    return {
      results: null,
      id: null,
      covers: null,
      currentId: null,
      access_token: "",
    };
  },
  methods: {
    searchIgdb(id) {
      this.currentId = id;

      const clientID = secrets.igdb.clientID;
      const token = secrets.igdb.token;
      let access_token;
      // api call
      let myHeadersaccess_token = new Headers();
      // myHeaders.append("", clientID);
      myHeadersaccess_token.append("Client-ID", clientID);
      myHeadersaccess_token.append("Authorization", `Bearer ${token}`);
      myHeadersaccess_token.append("Content-Type", "text/plain");

      var requestOptions = {
        method: "POST",
        headers: myHeadersaccess_token,
        redirect: "follow",
        // body: data,
      };
      // Get access_token
      fetch(
        `https://id.twitch.tv/oauth2/token?client_id=${clientID}&client_secret=${token}&grant_type=client_credentials`,
        requestOptions,
      )
        .then((response) => response.text())
        .then((result) => {
          const t = JSON.parse(result).access_token;
          console.warn(result);
          console.warn(t);
          this.access_token = t;

          this.getGames(id);
        })
        .catch((error) => console.log("error", error));


    },
    getGames(id) {
      let url = `https://api.igdb.com/v4/games/`;
      this.currentId = id;

      const game = this.results.find((item) => item.id === id).properties.Name
        .title[0].plain_text;
      const data = `search "${game}"; limit 5; fields name,cover.*;`;

      const clientID = secrets.igdb.clientID;
      console.warn(this.access_token);
      // api call
      let myHeaders = new Headers();
      myHeaders.append("Client-ID", clientID);
      myHeaders.append("Authorization", `Bearer ${this.access_token}`);
      // myHeaders.append("Content-Type", "text/plain");

      var requestOptions = {
        method: "POST",
        headers: myHeaders,
        redirect: "follow",
        body: data,
      };
      fetch(url, requestOptions)
        .then((response) => response.text())
        .then((result) => {
          console.warn(result);
          this.covers = JSON.parse(result);
          // console.warn(result);
        })
        .catch((error) => console.log("error", error));
    },
    update(cover) {
      cover = "https:" + cover.replace("thumb", "screenshot_huge");

      const token = secrets.notion.token;
      const pageId = this.currentId;
      let url = `https://api.notion.com/v1/pages/${pageId}`;

      let myHeaders = new Headers();
      myHeaders.append("Authorization", `Bearer ${token}`);
      myHeaders.append("Content-Type", "application/json");
      myHeaders.append("Notion-Version", "2021-08-16");

      const body = JSON.stringify({
        cover: {
          type: "external",
          external: {
            url: cover,
          },
        },
      });

      const requestOptions = {
        method: "PATCH",
        headers: myHeaders,
        redirect: "follow",
        body: body,
      };

      fetch(url, requestOptions)
        .then((response) => response.text())
        .then((result) => {
          console.warn("✅ Success");
          this.next();
        })
        .catch((error) => console.log("error", error));
    },
    next() {
      //remove current item from results
      this.results = this.results.filter((item) => item.id !== this.currentId);
      this.currentId = null;
      this.covers = null;
      this.searchIgdb(this.results[0].id);
    },
  },
  mounted() {
    const token = secrets.notion.token;
    const databaseId = secrets.notion.databaseId;
    let url = `https://api.notion.com/v1/databases/${databaseId}/query`;

    let myHeaders = new Headers();
    myHeaders.append("Authorization", `Bearer ${token}`);
    myHeaders.append("Content-Type", "application/json");
    myHeaders.append("Notion-Version", "2021-08-16");

    var requestOptions = {
      method: "POST",
      headers: myHeaders,
      redirect: "follow",
    };

    fetch(url, requestOptions)
      .then((response) => response.text())
      .then((result) => {
        const filter = JSON.parse(result).results.filter((item) => {
          return item.cover === null;
        });
        this.results = filter;
        if (JSON.parse(result).has_more) {
          more(JSON.parse(result).next_cursor);
        }
      })
      .catch((error) => console.log("error", error));

    const more = (cursor) => {
      console.warn("load more");

      const body = JSON.stringify({
        start_cursor: cursor,
      });

      var requestOptions = {
        method: "POST",
        headers: myHeaders,
        redirect: "follow",
        body: body,
      };

      fetch(url, requestOptions)
        .then((response) => response.text())
        .then((result) => {
          // console.warn(JSON.parse(result).next_cursor);
          const filter = JSON.parse(result).results.filter((item) => {
            return item.cover === null;
          });
          this.results = [...filter, ...this.results];
          if (JSON.parse(result).has_more) {
            more(JSON.parse(result).next_cursor);
          }
        })
        .catch((error) => console.log("error", error));
    };
  },
};
</script>

<template>
  <main>
    <div class="games">
      <div class="item" v-for="(item) in results" :key="item.id">
        <h3>{{ item.properties.Name.title[0].plain_text }}</h3>
        {{ item.id }}
        <button @click="searchIgdb(item.id)">Search cover</button>
      </div>
    </div>
    <div class="cover">
      <div class="item" v-for="(item) in covers" :key="item.id">
        <img v-if="item.cover" :src="item.cover.url.replace('thumb', 'screenshot_huge')" alt="">
        <button @click="update(item.cover.url)">Search cover</button>
      </div>
    </div>
  </main>
</template>


<style scoped>
main {
  display: flex;
  min-height: 100vh;
}

main>* {
  flex-basis: 100%;
}

.games {
  max-width: 500px;
  height: 100vh;
  overflow-y: scroll;
}

.games .item {
  background-color: rgba(0, 0, 0, 0.6);
}

.cover {
  max-width: 400px;
  height: 100vh;
  overflow-y: scroll;
}

.cover img {
  width: 100%;
}
</style>