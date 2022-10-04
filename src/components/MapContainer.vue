<script>
export default {
  data() {
    return { addressWithLatLang: [], map: null };
  },
  methods: {
    loadMap() {
      if (this.map) {
        this.map.remove();
      }

      this.map = L.map("map").setView([23.810332, 90.4125181], 8);
      
      L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(this.map);


      for (let i = 0; i < this.addressWithLatLang.length; i++) {
        // L.marker([this.addressWithLatLang[i].Latitude, this.addressWithLatLang[i].Longitude])
        //   .addTo(map)
        //   .bindPopup(this.addressWithLatLang[i].address);

        let circle = L.circle(
          [
            this.addressWithLatLang[i].Latitude,
            this.addressWithLatLang[i].Longitude,
          ],
          {
            color: "red",
            fillColor: "#f03",
            fillOpacity: 0.5,
            radius: this.addressWithLatLang[i].radius*100,
          }
        ).addTo(this.map);

        circle.bindPopup(
          `Total Patient ${this.addressWithLatLang[i].radius} from ${this.addressWithLatLang[i].address}`
        );
      }
    },

    async getData() {
      const resLanLng = await fetch("./locationWithLatAndLng.json");
      const resLanLngData = await resLanLng.json();

      const resAddress = await fetch("./address.json");
      const addressData = await resAddress.json();

      if (resLanLngData.length > 0 && addressData.length > 0) {
        addressData.forEach((address) => {
          const region = resLanLngData.find(
            (location) => location.address === address.address
          );

          if (region) {
            const existingAddress = this.addressWithLatLang.find(
              (location) => location.address === region.address
            );

            if (existingAddress) {
              const exixtingItemIndex = this.addressWithLatLang.findIndex(
                (location) => location.address === region.address
              );
              existingAddress.radius = existingAddress.radius + 1;

              this.addressWithLatLang[exixtingItemIndex] = existingAddress;
            } else {
              const newObj = {
                address: region.address,
                Latitude: region.Latitude,
                Longitude: region.Longitude,
                radius: 1,
              };

              this.addressWithLatLang.push(newObj);
            }
          }
        });
      }
    },
  },
  watch: {
    addressWithLatLang: {
      handler(newVal) {
        this.loadMap();
      },
      deep: true,
    },
  },
  mounted() {
    this.getData();
    this.loadMap();
  },
};
</script>

<template>
  <div class="container">
    <div id="map"></div>
  </div>
</template>

<style scoped>
.container {
  width: 800px;
}
#map {
  height: 580px;
}
</style>
