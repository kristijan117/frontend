<template>
<div>
    <section id="cart_items">
		<div class="container">
            <br><br>
			<div class="table-responsive cart_info">
				<table class="table table-condensed">
					<thead>
						<tr class="cart_menu">
							<td class="image"></td>
							<td class="price">Količina</td>
							<td class="quantity">Cijena</td>
							<!--<td class="total">Briši stanje košarice</td>-->
							<td></td>
						</tr>
					</thead>
					<tbody>
						<tr style="margin-top:20px">
							<td class="cart_description">
								<h4><a href="">Vaša košarica: </a></h4>
							</td>
							<td class="cart_quantity">
								<div class="cart_quantity_button">
									{{cartQty}}
								</div>
							</td>
							<td class="cart_total">
								<p class="cart_total_price">{{cartPrice}} €</p>
							</td>
							<!--<td class="cart_delete">
								<a style="margin-left:50px" class="cart_quantity_delete" href=""><i class="fa fa-times"></i></a>
							</td>-->
						</tr>
					</tbody>
				</table>
			</div>
		</div>
	</section> <!--/#cart_items-->

	<section id="do_action">
		<div class="container">
			<div class="heading">
				<h3>Nastavite s kupnjom</h3>
			</div>
			<div class="row">
				<div class="col-sm-6">
					<div class="chose_area">
						
						<ul class="user_info">
							<li class="single_field">
								<label>Država:</label>
								<select v-model="drzava">
									<option>Hrvatska</option>
									<option>Bosna i Hercegovina</option>
									<option>Srbija</option>
									<option>Crna Gora</option>
									<option>Austrija</option>
									<option>Njemačka</option>
									<option>Poljska</option>
									<option>Ukrajina</option>
								</select>
								
							</li>
							<li class="single_field zip-field">
								<label>Grad:</label>
								<input v-model="grad" type="text">
							
							</li>
							<li class="single_field zip-field">
								<label>Poštanski broj:</label>
								<input v-model="postanski_broj" type="text">
							</li>
						</ul>
					</div>
				</div>
				<div class="col-sm-6">
					<div class="total_area">
						<ul>
							<li>Ukupna potrošnja iz košarice<span>{{cartPrice}} €</span></li>
							<li>Porez <span>{{cartTax.toFixed(2)}} €</span></li>
							<li>Dostava <span>Besplatno</span></li>
							<li>Ukupno<span>{{cartPrice}} €</span></li>
						</ul>
							<a class="btn btn-default check_out" @click="confirmData()">Kupi</a>
							<div v-if="warnMessage" class="error-msg">
							<i class="fa fa-times-circle"></i>
							Ispunite sve podatke!
							</div>
							<div v-if="notLogedUser" class="error-msg">
							<i class="fa fa-times-circle"></i>
							Molimo vas prijavite se da biste mogli izvršiti kupnju!
							</div>
							<div v-if="successPaymentMessage" class="success-msg">
							<i class="fa fa-check"></i>
							Uspješno ste poslali zahtjev za plaćanje!
							</div>
					</div>
				</div>
			</div>
		</div>
        
	</section><!--/#do_action-->
    <Footer />
</div>
</template>

<script>
// @ is an alias to /src
import Footer from '@/components/footer.vue'
import { Cart, Auth, Payment } from '@/services'
import store from '@/store'

export default {
  name: 'Home',
  components: {
	Footer
  },
  data(){
	return {
		store:'',
		auth: Auth.state,
		cartItems: [],
		cartPrice: 0,
		cartQty: 0,
		cartTax: 0,
		drzava: "",
		grad:"",
		postanski_broj:"",
		warnMessage:false,
		notLogedUser:false,
		successPaymentMessage:false
	}
  },
  methods:{
	async cartData(){
			let dohvat = await Cart.fetchCart();
			this.cartItems = JSON.parse(JSON.stringify(dohvat.cart));
			this.cartPrice = this.cartItems.ukupnaCijena.toFixed(2);
			this.cartQty = this.cartItems.ukupnaKol;
			this.cartTax = this.cartPrice*0.05;
		},
	async confirmData(){
		if(this.cartPrice !=="" && this.cartQty !=="" && this.drzava !=="" && this.grad !=="" && this.postanski_broj !=="" && this.auth.authenticated==true){
		let placanje = {
                cijena: this.cartPrice,
                kolicinaProizvoda:this.cartQty,
				drzava: this.drzava,
				grad: this.grad,
				postanski_broj: this.postanski_broj,
				korisnik: this.auth.userEmail
            }
		this.successPaymentMessage = true;
		Payment.pay(placanje);
		}
		else if(this.auth.authenticated==false){
			this.notLogedUser=true;
		}
		else{
			this.warnMessage=true;
		}
	}
  },
  created(){
		this.cartData()
  }
}
</script>

<style scoped>
@import url('//maxcdn.bootstrapcdn.com/font-awesome/4.2.0/css/font-awesome.min.css');

.info-msg,
.success-msg,
.warning-msg,
.error-msg {
  margin: 10px 0;
  padding: 10px;
  border-radius: 3px 3px 3px 3px;
}
.info-msg {
  color: #059;
  background-color: #BEF;
}
.success-msg {
  color: #270;
  background-color: #DFF2BF;
}
.warning-msg {
  color: #9F6000;
  background-color: #FEEFB3;
}
.error-msg {
  color: #D8000C;
  background-color: #FFBABA;
}
</style>