<template>
    <div class="content w-100 h-50 p-3">
        <h1 class="title p-3">Parking API</h1>
        <div v-show="alert" class="alert alert-danger" role="alert" id="alert">
            Message
        </div>
        <div class="row d-flex ms-1">
            <div class="card w-25 p-2 m-2">
                <h5>Check-In - Registrar novo veículo</h5>
                <label class="form-label" for="placa">Placa do veículo</label>
                <div class="d-flex">
                    <input class="form-control me-2" id="placa" type="text" placeholder="digite a placa do veículo" v-model="plate">
                    <button class="btn btn-success" type="submit" @click="newVehicle()">Enviar</button>
                </div>
            </div>
            <div v-show="checkout" class="card w-25 p-2 m-2">
                <h5>Check-Out - Pagamento</h5>
                <p>Placa: {{ checkoutData.plate }}</p>
                <p>Total a pagar: <strong>{{ checkoutData.value }}</strong></p>
                <div class="d-flex justify-content-end">
                    <button class="btn btn-primary" type="submit" @click="finishCheckout">Finalizar</button>
                </div>
            </div>
        </div>
        <div class="card p-3 m-2">
            <div class="col d-flex justify-content-between">
                <h5>Status do estacionamento</h5>
                <strong>Total de veículos: {{ usedSpots }}</strong>
            </div>
            <div class="col d-flex flex-wrap">
                <div v-for="s in spots" :key="s.id" class="card mb-2 me-2 p-2 d-flex flex-row justify-content-between spot-card">
                    <div class="column me-3">
                        <strong>ID:{{ s.id }}</strong><br>
                        Placa: {{ s.plate }}<br>
                        Entrada: {{ s.entryTime }}<br>
                        <span> {{ s.status }} </span>
                    </div>
                    <div class="column d-flex flex-column justify-content-end">
                        <button class="btn btn-outline-danger" type="submit" @click="deleteVehicle(s.plate)">Saída</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            spots: null,
            usedSpots: 0,
            plate: "",
            alert: false,
            checkout: false,
            checkoutData: {
                plate: "",
                value: 0.0
            }
        }
    },
    methods: {
        getById(id) {
            return document.getElementById(id);
        },
        async finishCheckout() {
            this.checkoutData.plate = "";
            this.checkoutData.value = 0.0;
            this.checkout = false;
            this.messageAlert("Pagamento realizado!", 1);
        },
        async checkOut(data) {
            this.checkoutData.plate = data.plate;
            this.checkoutData.value = data.payment.toLocaleString('pt-br',{style: 'currency', currency: 'BRL'});
            this.checkout = true;
        },
        async messageAlert(msg, code) {
            // code 0: error
            // code 1: sucess
            if (msg != "") {
                var messageAlert = this.getById("alert");
                messageAlert.innerHTML = msg;
                switch (code) {
                    case 0:
                        messageAlert.classList.remove("alert-sucess");
                        messageAlert.classList.add("alert-danger");
                        break;
                    case 1:
                        messageAlert.classList.remove("alert-danger");
                        messageAlert.classList.add("alert-success");
                        break;
                    default:
                        break;
                }
                this.alert = true;
            } else {
                this.alert = false;
            }

            setTimeout(() => {this.alert = false}, 6500);
        },
        async newVehicle() {

            if (this.plate != "") {
                const header = new Headers({
                    "Content-Type": "application/json",
                    "Accept": "*/*"
                });

                var data = `{"plate":"${this.plate}"}`;

                const params = {
                    method: 'POST',
                    headers: header,
                    body: data
                };

                const request = await fetch("http://localhost:8080/parking", params);

                if (request.status == 200) {
                    this.messageAlert("Veículo registrado!", 1);
                    this.getVehicles();
                } else {
                    const response = await request.json();
                    this.messageAlert(response.message, 0);
                }
            } else {
                this.messageAlert("Informe uma placa!", 0);
            }
        },
        async deleteVehicle(plate) {
            if (plate != "") {
                const header = new Headers({
                    "Content-Type": "application/json",
                    "Accept": "*/*"
                });

                var data = `{"plate":"${plate}"}`;

                const params = {
                    method: 'PUT',
                    headers: header,
                    body: data
                };

                const request = await fetch("http://localhost:8080/parking", params);

                if (request.status == 200) {
                    this.messageAlert("Check-out realizado com sucesso!", 1);
                    this.checkOut(await request.json());
                    this.getVehicles();
                } else {
                    const response = await request.json();
                    this.messageAlert(response.message, 0);
                }
            }
        },
        async getVehicles() {
            const request = await fetch("http://localhost:8080/parking");
            this.spots = await request.json();
            this.usedSpots = this.spots.length;
        },
    },
    mounted() {
        this.getVehicles();
    }
}
</script>

<style>
.alert {
    width: 500px;
}

.spot-card {
    width: 25rem;
}
</style>