<template>
  <div v-if="loading==true" class="loader"></div>

  <div v-else class="tabs content__tabs" style="margin-top:-20px">
    <vue-progress-bar></vue-progress-bar>
    <ul class="nav nav--full-width nav-fill nav-tabs" id="weekTab" role="tablist">
      <li :key="day.uuid" v-for="(day,index) in componentData.menuDays" class="nav-item">
        <a
          class="nav-link"
          :class="{active: index==0}"
          :id="`${day.dayOfWeek.name}-tab`"
          data-toggle="tab"
          :href="`#${day.dayOfWeek.name}`"
          role="tab"
          :aria-controls="day.dayOfWeek.name"
          aria-selected="false"
        >{{day.dayOfWeek.name}}</a>
      </li>
    </ul>
    <div class="tab-content" id="weekTabContent">
      <div
        :key="day.uuid"
        v-for="(day,index) in componentData.menuDays"
        class="tab-pane fade"
        :class='{"show active" : index==0}'
        :id="day.dayOfWeek.name"
        role="tabpanel"
        :aria-labelledby="day.dayOfWeek.name"
      >

        <template v-if='day.menuMeals.length>0'> <!-- v-if='meal.menuPositions.length>0' -->
          <div v-for="meal in day.menuMeals" :key="meal.uuid"  class="container-fluid mt-3">
            <div class="row">
              <div class="col-md-12">
                <div class="media">

                  <a href="#" class="dropdown-menu-gallery-button" v-bind:id="'dropdownMenuButton-'+meal.uuid" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" @click="onClickDropdown">
                    <i class="h3 fe fe-zoom-in dropdown-menu-zoom"></i>
                    <i class="text-white bg-primary small fe fe-camera dropdown-menu-camera" v-if="meal.image"></i>
                    <img v-if="meal.dMeal.icon" class="rounded-circle" alt="" :src="meal.dMeal.icon" />
                    <img v-else class="rounded-circle" alt="" src="/build/static/images/no_image.png"  style="max-height:55px" />
                  </a>

                  <!-- dropdown gallery -->
                  <div class="dropdown-menu dropdown-menu-gallery p-0" v-bind:aria-labelledby="'dropdownMenuButton-'+meal.uuid">
                    <div class="card border-0" style="max-width: 400px;">
                      <div class="card-header d-flex bg-white" v-if="meal.image">
                        <a target="_blank" rel="noopener noreferrer" :href="'/media/'+ meal.image.contentUrl" id="external-btn" class="btn btn-sm btn-outline-primary px-2" :class='{"imageNameCentered": !canEdit}'><i class="fe fe-external-link mr-2"></i>{{meal.image.contentUrl}}</a>
                        <button v-if='canEdit' @click="deleteMealImage(meal)" id="delete-btn" class="btn btn-sm btn-outline-danger ml-auto px-2"><i class="fe fe-trash-2 mr-2"></i>Usuń</button>
                      </div>
                      <div class="bg-light d-flex position-relative" style="min-height:200px; max-height:300px; overflow:auto">
                        <img v-if="imageLoad==true" src="/build/static/images/mini_loader.gif" class="image-mini-loader" />
                        <template v-if="meal.image">
                          <img :src="'/media/'+ meal.image.contentUrl" class="d-block mx-auto" style="max-width:400px; object-fit: contain;" :alt="meal.image.contentUrl"> <!-- m-auto -->
                        </template>
                        <template v-else>
                          <img src="/build/static/images/no_image.png" class="d-block mx-auto" style="max-width:400px; object-fit: contain;" alt="No image available"> <!-- m-auto -->
                        </template>
                      </div>
                      <!--
                      <nav class="mt-n3">
                        <ul class="pagination pagination-sm justify-content-center mb-0">
                          <li class="page-item disabled"><button type="button" class="page-link"><i class="fe fe-chevron-left"></i></button></li>
                          <li class="page-item"><button type="button" class="page-link"><i class="fe fe-chevron-right"></i></button></li>
                        </ul>
                      </nav>
                      -->
                      <div v-if='canEdit' class="card-body">
                        <h5 class="card-title">Zmień zdjęcie [{{meal.dMeal.name}}]</h5>
                        <div class="input-group">
                          <div class="custom-file">
                            <input type="file" :name="'file'+meal.uuid" :id="'file'+meal.uuid" :ref="'file'+meal.uuid" accept="image/*" @change="onFileChanged" class="custom-file-input" />
                            <label class="custom-file-label" :for="'file'+meal.uuid" data-browse="Przeglądaj">
                              {{ fileName ? fileName : 'Wybierz plik'}}
                            </label>
                          </div>
                          <div class="input-group-append">
                            <button v-bind:id="'btn-upload-'+meal.uuid" :disabled="file== ''" class="btn btn-primary" style="position:relative; z-index:100" @click="onUpload(meal)"><i class="fe fe-save mr-2"></i>Zapisz</button>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                  <!-- /dropdown gallery -->

                  <div class="media-body">
                    <h3 class="ml-3 mt-3">{{meal.dMeal.name}}</h3>
                    <img
                      v-if="canEdit"
                      src="/build/static/images/icons/actions-general/cross-grey.png"
                      alt
                      @click="deleteWholeMeal(day,meal)"
                      title="Usuwa cały posiłek"
                    >
                  </div>
                </div>
              </div>
            </div>
            <div class="table-responsive mt-2 table--menu">
              <table v-if='componentData.isActive == false' class="table table--grid table-sm table-striped">
                <thead>
                  <tr>
                    <th class="w-25"></th>
                    <th colspan="2" style="width:15%"></th>
                    <th colspan="10" class="bg-light" style="border: 1px solid #cdcdcd;">Wartości na porcję</th>
                    <th></th>
                  </tr>
                  <tr>
                    <th>Produkt</th>
                    <th colspan="2">Ilość g na porcję <i class="fe fe-info text-warning ml-1" data-placement="top" data-toggle="tooltip" title="Przykładowo, bułka na magazynie ma 43g na szt., więc aby wydać do posiłku 2 bułki, należy wpisać 86g."></i></th>
                    <th colspan="2" class="border-left">Energia [kcal]</th>
                    <th colspan="2" class="border-left">Tłuszcz [g]</th>
                    <th colspan="2" class="border-left">Białko [g]</th>
                    <th colspan="2" class="border-left">Węglowodany [g]</th>
                    <th colspan="2" class="border-left border-right">Cena <i class="fe fe-info text-warning ml-1" data-placement="top" data-toggle="tooltip" title="Aby poznać ceny, należy wybrać kontekst kuchni. Jeżeli ceny mimo wszystko się nie wyświetlają, może to oznaczać brak surowców w danej kuchni"></i></th>
                    <th v-if="canEdit" class="text-center">Akcje</th>
                  </tr>
                </thead>
                <tbody>
                  <!-- ALREADY ADDED PRODUCTS -->
                  <template v-if='meal.menuPositions.length>0'>
                      <tr v-for='position in meal.menuPositions' :key='position.uuid'>
                          <td class="w-25">
                              <span v-if='position.material'>{{position.material.name}}</span>
                              <span v-else>{{position.recipe.name}}</span>
                              <template v-if="position.material"><span class="small text-muted text-right">{{position.material.amountOfGramPerStorageUnit}}g/{{position.material.unit.shortcut}}</span></template>
                          </td>
                          <td class="text-right" style="width:1%;white-space:nowrap;">{{position.quantity}}</td>
                          <td class="text-grey small">g</td>

                          <td class="text-right">{{valuesPerPortion(position, 'energyInKcal',day,meal)}}</td>
                          <td class="text-grey small">kcal</td>
                          <td class="text-right">{{valuesPerPortion(position, 'fat',day,meal)}}</td>
                          <td class="text-grey small">g</td>
                          <td class="text-right">{{valuesPerPortion(position, 'protein',day,meal)}}</td>
                          <td class="text-grey small">g</td>
                          <td class="text-right">{{valuesPerPortion(position, 'carbohydrates',day,meal)}}</td>
                          <td class="text-grey small">g</td>
                          <td class="text-right">
                              <template v-if="position.price !== null">
                                  {{ position.price.toFixed(2) }}
                              </template>
                              <template v-else>Brak</template>
                          </td>
                          <td class="text-grey small">{{ position.price !== null ? 'zł' : '' }}</td>
                          <td class='menu-action' v-if="canEdit">
                              <a @click="deletePosition(position.uuid, index, day, meal)" style="cursor:pointer"><i class="fe fe-trash-2 text-danger"></i></a>
                          </td>
                          <td v-else class='menu-action'></td>
                      </tr>
                  </template>
                  <!-- CURRENTLY EDITTED PRODUCT -->
                  <tr v-if="meal.editMode" class="bg-secondary">
                    <td class="w-25">
                      <select-materials
                        :materials="true"
                        :recipes="true"
                        :onSelectedMaterial="onSelectMaterial"
                        :dayData='{"day": day,"meal": meal}'
                      ></select-materials>
                    </td>
                    <td colspan="2"> <!-- productQuantity -->
                      <input
                        v-if='selectedProduct[meal.uuid].productName && selectedProduct[meal.uuid].productName.item["@type"]!=="Recipe"'
                        class="form__input w-100"
                        value="xxx" :placeholder="selectedProduct[meal.uuid].productName.item == null ? 'Ilość g na porcję' : 'Ilość g na porcję '+selectedProduct[meal.uuid].productName.item.amountOfGramPerStorageUnit+'g/'+selectedProduct[meal.uuid].productName.item.unit.shortcut"
                        v-model="selectedProduct[meal.uuid].productQuantity"
                      >
                      <input
                        v-if='selectedProduct[meal.uuid].productName && selectedProduct[meal.uuid].productName.item["@type"]=="Recipe"'
                        type="text" class="form__input w-100" placeholder="Wpisz ilość gramów"
                        :value="selectedProduct[meal.uuid].productName.item.portionValue"
                        disabled
                      >
                    </td>

                    <td class="text-right"> <!-- energyInKcal -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.energyInKcal * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.energyInKcal)
                        }}
                      </template>
                    </td>
                    <td class="text-grey small">kcal</td>
                    <td class="text-right"> <!-- fat -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.fat * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.fat)
                        }}
                      </template>
                    </td>
                    <td class="text-grey small">g</td>
                    <td class="text-right"> <!-- protein -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.protein * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.protein)
                        }}
                      </template>
                    </td>
                    <td class="text-grey small">g</td>
                    <td class="text-right"> <!-- carbohydrates -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.carbohydrates * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.carbohydrates)
                        }}
                      </template>
                    </td>
                    <td class="text-grey small">g</td>
                    <td class="text-right"> <!-- price -->

                    </td>
                    <td class="text-grey small">zł</td>
                    <td></td>
                  </tr>

                  <!-- ADD BUTTON'S LINE -->
                  <tr style="border-top:4px solid #dee2e6;">
                    <td v-if="canEdit" class="border-bottom-0 w-25 bg-white">
                      <div v-if="meal.editMode == false">
                        <button 
                          name="plus-position"
                          @click="switchEditMode(day, meal)"
                          class="btn btn-outline-primary btn-block p-0"
                        ><i class="fe fe-plus-square mr-2"></i>Dodaj</button>
                      </div>

                      <div v-else style='display: flex; justify-content: space-between;'>
                        <button
                          name="save-position"
                          class="btn btn-outline-primary p-0"
                          style='width:80%'
                          @click="saveNewProduct(day,meal)"
                        ><i class="fe fe-save mr-2"></i>Zapisz</button>

                        <button style='width:15%' class="btn btn-outline-danger p-0" @click='closeUnsavedPosition(day, meal)'>X</button>
                      </div>
                    </td>
                    <td v-else class="border-bottom-0 w-25 bg-white"></td>


                    <td colspan="2" class="border-bottom-0 font-weight-bold bg-white">Podsumowanie</td>  <!--  {{meal.dMeal.name}}  -->
                    <!-- <td class="border-bottom-0">g</td> -->

                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'energyInKcal')}} </td>
                    <td class="border-bottom-0 text-grey small bg-white">kcal</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'fat')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">g</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'protein')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">g</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'carbohydrates')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">g</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white" >{{mealSummPerPortion(day,meal,'price')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">zł</td>
                    <td class="border-bottom-0 bg-white"></td>
                  </tr>
                </tbody>
              </table>

              <table v-else-if='componentData.isActive == true && meal.menuPositions.length==0' class="table table--grid table-sm">
                  <tr><h5 style='color: #11080e;'>Brak pozycji w danym posiłku</h5></tr>
              </table>

              <table v-else class="table table--grid table-sm table-striped">
                <thead>
                  <tr>
                    <th class="w-25"></th>
                    <th colspan="2" style="width:15%"></th>
                    <th colspan="10" class="bg-light" style="border: 1px solid #cdcdcd;">Wartości na porcję</th>
                    <th v-if="canEdit"></th>
                  </tr>
                  <tr>
                    <th>Produkt</th>
                    <th colspan="2">Ilość g na porcję <i class="fe fe-info text-warning ml-1" data-placement="top" data-toggle="tooltip" title="Przykładowo, bułka na magazynie ma 43g na szt., więc aby wydać do posiłku 2 bułki, należy wpisać 86g."></i></th>
                    <th colspan="2" class="border-left">Energia [kcal]</th>
                    <th colspan="2" class="border-left">Tłuszcz [g]</th>
                    <th colspan="2" class="border-left">Białko [g]</th>
                    <th colspan="2" class="border-left">Węglowodany [g]</th>
                    <th colspan="2" class="border-left border-right">Cena <i class="fe fe-info text-warning ml-1" data-placement="top" data-toggle="tooltip" title="Aby poznać ceny, należy wybrać kontekst kuchni. Jeżeli ceny mimo wszystko się nie wyświetlają, może to oznaczać brak surowców w danej kuchni"></i></th>
                    <th v-if="canEdit">Akcje</th>
                  </tr>
                </thead>
                <tbody>
                  <!-- ALREADY ADDED PRODUCTS -->
                  <template v-if='meal.menuPositions.length>0'>
                      <tr v-for='position in meal.menuPositions' :key='position.uuid'>
                          <td class="w-25">
                              <span v-if='position.material'>{{position.material.name}}</span>
                              <span v-else>{{position.recipe.name}}</span>
                              <template v-if="position.material"><span class="small text-muted text-right">{{position.material.amountOfGramPerStorageUnit}}g/{{position.material.unit.shortcut}}</span></template>
                          </td>
                          <td class="text-right" style="width:1%;white-space:nowrap;">{{position.quantity}}</td>
                          <td class="text-grey small">g</td>

                          <td class="text-right">{{valuesPerPortion(position, 'energyInKcal',day,meal)}}</td>
                          <td class="text-grey small">kcal</td>
                          <td class="text-right">{{valuesPerPortion(position, 'fat',day,meal)}}</td>
                          <td class="text-grey small">g</td>
                          <td class="text-right">{{valuesPerPortion(position, 'protein',day,meal)}}</td>
                          <td class="text-grey small">g</td>
                          <td class="text-right">{{valuesPerPortion(position, 'carbohydrates',day,meal)}}</td>
                          <td class="text-grey small">g</td>
                          <td class="text-right">
                              <template v-if="position.price !== null">
                                  {{ position.price.toFixed(2) }}
                              </template>
                              <template v-else>Brak</template>
                          </td>
                          <td class="text-grey small">{{ position.price !== null ? 'zł' : '' }}</td>
                          <td class='menu-action' v-if="canEdit">
                              <a @click="deletePosition(position.uuid, index, day, meal)" style="cursor:pointer"><i class="fe fe-trash-2 text-danger"></i></a>
                          </td>
                      </tr>
                  </template>
                  <!-- CURRENTLY EDITTED PRODUCT -->
                  <tr v-if="meal.editMode">
                    <td class="w-25">
                      <select-materials
                        :materials="true"
                        :recipes="true"
                        :onSelectedMaterial="onSelectMaterial"
                        :dayData='{"day": day,"meal": meal}'
                      ></select-materials>
                    </td>
                    <td> <!-- productQuantity -->
                      <input
                        v-if='selectedProduct[meal.uuid].productName && selectedProduct[meal.uuid].productName.item["@type"]!=="Recipe"'
                        class="form__input"
                        value="xxx"
                        v-model="selectedProduct[meal.uuid].productQuantity"
                      >
                      <input
                        v-if='selectedProduct[meal.uuid].productName && selectedProduct[meal.uuid].productName.item["@type"]=="Recipe"'
                        type="text" class="form__input" placeholder="Wpisz ilość gramów"
                        :value="selectedProduct[meal.uuid].productName.item.portionValue"
                        disabled
                      >
                    </td>
                    <td> <!-- energyInKcal -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.energyInKcal * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.energyInKcal)
                        }} <span class="text-grey small">kcal</span>
                      </template>
                    </td>
                    <td> <!-- fat -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.fat * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.fat)
                        }} <span class="text-grey small">g</span>
                      </template>
                    </td>
                    <td> <!-- protein -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.protein * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.protein)
                        }} <span class="text-grey small">g</span>
                      </template>
                    </td>
                    <td> <!-- carbohydrates -->
                      <template v-if="selectedProduct[meal.uuid].productName != ''">
                        {{
                        selectedProduct[meal.uuid].productName.item['@type'] == 'Material' ?
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.carbohydrates * ( selectedProduct[meal.uuid].productQuantity - ( selectedProduct[meal.uuid].productQuantity * selectedProduct[meal.uuid].productName.item.nutritionalValuesAssigned.wastePercent/100 ) ) / 100 ).toFixed(2)
                        :
                        (selectedProduct[meal.uuid].productName.item.nutritionalValuesAssignedSum.carbohydrates)
                        }} <span class="text-grey small">g</span>
                      </template>
                    </td>
                    <td class="text-right"> <!-- price -->

                    </td>
                    <td class="text-grey small">zł</td>
            
                  </tr>

                  <!-- ADD BUTTON'S LINE -->
                  <tr style="border-top:4px solid #dee2e6;">
                    <td v-if="canEdit" class="border-bottom-0 w-25">
                      <div v-if="meal.editMode == false">
                        <button 
                          name="plus-position"
                          @click="switchEditMode(day, meal)"
                          class="btn btn-outline-primary btn-block p-0"
                        >+</button>
                      </div>

                      <div v-else style='display: flex; justify-content: space-between;'>
                        <button
                          name="save-position"
                          class="btn btn-outline-primary p-0"
                          style='width:80%'
                          @click="saveNewProduct(day,meal)"
                        >Zapisz</button>

                        <button style='width:15%' class="btn btn-outline-danger p-0" @click='closeUnsavedPosition(day, meal)'>X</button>
                      </div>
                    </td>
                    <td v-else class="border-bottom-0 w-25"></td>


                    <td colspan="2" class="border-bottom-0 font-weight-bold bg-white">Podsumowanie</td>  <!--  {{meal.dMeal.name}}  -->
                    <!-- <td class="border-bottom-0">g</td> -->

                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'energyInKcal')}} </td>
                    <td class="border-bottom-0 text-grey small bg-white">kcal</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'fat')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">g</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'protein')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">g</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{mealSummPerPortion(day,meal,'carbohydrates')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">g</td>
                    <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white" >{{mealSummPerPortion(day,meal,'price')}}</td>
                    <td class="border-bottom-0 text-grey small bg-white">zł</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </template>

        <!-- Total summ block -->
        <div v-if='day.menuMeals.length>0' class="container-fluid mt-3">
            <div class="table-responsive table--menu">
                <table v-if='checkMealsPositions(day.menuMeals)' class="table table--grid table-sm">
                    <thead>
                        <tr>
                            <th class="w-25"></th>
                            <th colspan="2" style="width:16%"></th>
                            <th colspan="10" class="bg-light" style="border: 1px solid #cdcdcd;">Wartości na dzień</th>
                            <th v-if="canEdit"></th>
                        </tr>
                        <tr>
                            <th></th>
                            <th colspan="2"></th>
                            <th colspan="2" class="border-left">Energia [kcal]</th>
                            <th colspan="2" class="border-left">Tłuszcz [g]</th>
                            <th colspan="2" class="border-left">Białko [g]</th>
                            <th colspan="2" class="border-left">Węglowodany [g]</th>
                            <th colspan="2" class="border-left border-right">Cena <i class="fe fe-info text-warning ml-1" data-placement="top" data-toggle="tooltip" title="Aby poznać ceny, należy wybrać kontekst kuchni. Jeżeli ceny mimo wszystko się nie wyświetlają, może to oznaczać brak surowców w danej kuchni"></i></th>
                            <th v-if="canEdit"></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr style="border-top:4px solid #dee2e6;">
                            <td  class="border-bottom-0 w-25 bg-white">
                                &nbsp;
                            </td>
                            <td colspan="2" class="border-bottom-0 font-weight-bold bg-white">Podsumowanie całości</td>

                            <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{summForWholeDayPortion(day,'energyInKcal')}} </td>
                            <td class="border-bottom-0 text-grey small bg-white">kcal</td>
                            <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{summForWholeDayPortion(day,'fat')}}</td>
                            <td class="border-bottom-0 text-grey small bg-white">g</td>
                            <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{summForWholeDayPortion(day,'protein')}}</td>
                            <td class="border-bottom-0 text-grey small bg-white">g</td>
                            <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white">{{summForWholeDayPortion(day,'carbohydrates')}}</td>
                            <td class="border-bottom-0 text-grey small bg-white">g</td>
                            <td style="width:1%;white-space:nowrap;" class="text-right border-bottom-0 bg-white" >{{summForWholeDayPortion(day,'price')}}</td>
                            <td class="border-bottom-0 text-grey small bg-white">zł</td>
                            <td v-if="canEdit" class="border-bottom-0"></td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <div v-if='day.menuMeals.length == 0 && currentStatus == true' class="container-fluid mt-4" style='margin-bottom: -1rem;'>
            <h3 style='text-align: center; margin:0 !important;'>Brak posiłków w danym dniu</h3>
        </div>
        <!-- Select type of meal block -->
        <div class="container-fluid mt-3">
          <div class="row">
            <div class="col-md-3">
              <div v-if="canEdit" class="media" style="align-items:center">
                <img class="rounded-circle mr-3" alt src="/build/static/images/new-dish.png">
                <div class="media-body">
                  <select
                    class="w-100 meal-select"
                    title="Dodaj kolejny posiłek"
                    v-model="currentSelectedMeal"
                    ref="selectMealSelector"
                  >
                      <option
                        class='meal-select__option'
                        v-for="mealType of typeOfMeal"
                        :disabled='checkIfMealExists(day,mealType)'
                        :key="mealType.uuid"
                        :value='{"typeOfMeal" : mealType,"selectedDay":day}'>
                        {{mealType.name}}
                      </option>
                  </select>
                </div>
              </div>
            </div>
            <div
              v-if="is_granted_menu_edit && currentStatus === false"
              class="col-md-9 text-right"
              style="display:flex; justify-content:flex-end; align-items:center"
            > 
              <button 
                  class="basic-info__add-btn" 
                  :class='{"meal-select--disabled": !isThereAnyPosition}'
                  @click="changeStatus">
                  {{ currentStatus === false ? 'Publikuj' : 'Wróć do Szkicu'}}
              </button> <!-- Zmień status -->
            </div>
            
            <div v-else 
                 class="col-md-9 text-right"
                 style="display:flex; justify-content:flex-end; align-items:center; font-weight: bold;">
        
                <span>Status:  {{currentStatus == true ? ' Opublikowany' : ' Szkic'}}</span>
            </div>

          </div>
        </div>
      </div>
    </div>
    <custom-alert></custom-alert>
    <div v-if="isUpdated" class='loader-container'>
        <div  class="loader loader--fixed"></div>
    </div>
  </div>
</template>


<script>
    import SelectMaterials from "./../SelectMaterials.vue";
    import Alert from '../Alert.vue';
    import {EventBus} from '../../app.js';
    import axios from "axios";
    import { isRegExp } from 'util';
    export default {
  props: ["endpoint", "menuid","is_granted_menu_edit"],
  components: {
    "select-materials": SelectMaterials,
    "custom-alert": Alert
  },
  data: function() {
    return {
      typeOfMeal: [],
      currentSelectedMeal: "",
      selectedProduct: {},
      currentStatus: false,
      componentData: null,
      loading: false,
      imageLoad: false,
      isThereAnyPosition: null,
      file: '',
      fileName: null,
      mealImage: null,
      mediaObjectId: null,
      isUpdated: false,
    };
  },

  created() {
    this.loading = true;
    this.$Progress.start();
    fetch(this.endpoint)
      .then(response => {
        return response.json();
      })
      .then(data => {
        this.componentData = {
          ...data,
          menuDays: data.menuDays.map(day => {
            return {
              ...day,
              menuMeals: day.menuMeals.map(meal => {
                return {
                  ...meal,
                  editMode: false,
                };
              })
            };
          })
        };
        this.currentStatus = data.isActive;
        EventBus.$emit('passMenuStatus', this.currentStatus);
        this.loading = false;
        this.$Progress.finish();
        //jquery tooltip
        this.$nextTick(function() {
            $('[data-toggle="tooltip"]').tooltip({ boundary: 'window' })
        })
      });


    fetch("/api/d_meals?context=dMeal&active_or_selected=null")
      .then(response => response.json())
      .then(data => {
        this.typeOfMeal = [...data["hydra:member"]];
      });
    EventBus.$on('closeUnsavedPosition',({day, meal})=>{
        this.closeUnsavedPosition(day,meal);
    })
  },
  updated(){
    let isThereAnyPosition = [];

    for(let day of this.componentData.menuDays){
      for(let menuMeal of day.menuMeals){
        if(menuMeal.menuPositions.length >0){
          isThereAnyPosition.push(true);
        }
      }
    }
    this.isThereAnyPosition = isThereAnyPosition.includes(true) ? true : false;
    //jquery tooltip
    this.$nextTick(function() {
        $('[data-toggle="tooltip"]').tooltip({ boundary: 'window' })
    })
  },
  computed: {
    canEdit(){
        return this.currentStatus == false && this.is_granted_menu_edit == 1;
    }
  },
  methods: {
    changeStatus() {
      if(this.isThereAnyPosition){
        this.$Progress.start();
        this.isUpdated = true;
        fetch(this.endpoint, {
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json"
          },
          method: "PUT",
          body: JSON.stringify({
            isActive: true
          })
        }).then(response => {
          (async ()=>{
              let data = await response.json();
              EventBus.$emit('triggerAlert', {
                  message: response.ok ? 'Status został zmieniony' : data, 
                  type:'changeStatus'
              });

              if(response.ok === true){
                  this.$Progress.finish();
                  this.currentStatus = true;
              }
              else {
                  this.$Progress.fail();
              }
              this.isUpdated = false;
          })()
        }).catch(err=>{
          this.isUpdated = false;
          this.$Progress.fail();
          EventBus.$emit('triggerAlert', {message: `Błąd serwera`, triggerManualError:true});
          console.error('Error:', error)
        });
      }
      else {
        EventBus.$emit('triggerAlert', {message: `Musisz mieć co najmniej 1 pozycję żeby zmienić status`, triggerManualError:true, type:'changeStatusError'});
      }
    },
    deleteWholeMeal: function(day, meal) {
      this.$Progress.start();
      this.isUpdated = true;
      fetch(`/api/menu_meals/${meal.uuid}`, {
        method: "DELETE"
      }).then(response => {
        (async ()=>{
            EventBus.$emit('triggerAlert', {
                message: response.ok ? `Usunięto: ${meal.dMeal.name}` : 'Usunięcie się nie powiodło', 
                type:'deleteWholeMeal'
            });

            if(response.ok === true){
                this.$Progress.finish();
                let linkToDay = this.componentData.menuDays.find(element => {
                  return element.uuid == day.uuid;
                });
                linkToDay.menuMeals.splice(linkToDay.menuMeals.indexOf(meal), 1);
            }
            else {
                this.$Progress.fail();
            }
            this.isUpdated = false;
        })()
      }).catch(err=>{
        this.isUpdated = false;
        EventBus.$emit('triggerAlert', {message: `Błąd serwera`, triggerManualError:true});
        console.error('Error:', error)
      });
    },
    onSelectMaterial(selectedObject, dayData) {
      this.selectedProduct = {
        ...this.selectedProduct,
        [dayData.meal.uuid]: {
          productName: {
            day: {
              ...dayData.day
            },
            meal: {
              ...dayData.meal
            },
            item: {
              ...selectedObject
            }
          },
          productQuantity: null
        }
      };
    },
    switchEditMode: function(day, meal) {
      let copyOfDaysOfWeak = [...this.componentData.menuDays];
      let currentDayIndex = copyOfDaysOfWeak.indexOf(day);
      let currentMealIndex = copyOfDaysOfWeak[currentDayIndex].menuMeals.indexOf(meal);

      let copyOfCurrentDay = {
        ...this.componentData.menuDays[currentDayIndex]
      };
      copyOfCurrentDay.menuMeals[currentMealIndex].editMode = !copyOfCurrentDay.menuMeals[currentMealIndex].editMode;

      copyOfDaysOfWeak.splice(currentDayIndex, 1, copyOfCurrentDay);
      this.componentData = {
        ...this.componentData,
        menuDays: [...copyOfDaysOfWeak]
      };

      this.selectedProduct = {
        ...this.selectedProduct,
        [meal.uuid]: {
          productName: "",
          productQuantity: null
        }
      };
    },
    saveNewProduct: function(day, meal) {
      let copyOfDaysOfWeak = [...this.componentData.menuDays];
      let currentDayIndex = copyOfDaysOfWeak.indexOf(day);
      let currentMealIndex = copyOfDaysOfWeak[currentDayIndex].menuMeals.indexOf(meal);
      let copyOfCurrentDay = { ...this.componentData.menuDays[currentDayIndex] };
      if(this.selectedProduct[meal.uuid].productName == "" || this.selectedProduct[meal.uuid].productQuantity == null && !this.selectedProduct[meal.uuid].productName.item["@type"] == "Recipe"){
        EventBus.$emit('triggerAlert', {message: `Wybierz pozycję i ustal ilość gramów na porcję`, triggerManualError:true, type:'positionAddingError'})
        return false;
      }
      if (this.selectedProduct[meal.uuid].productName.item["@type"] == "Material") {

        if (this.selectedProduct[meal.uuid].productQuantity == null || this.selectedProduct[meal.uuid].productQuantity.trim() === '') {
          EventBus.$emit('triggerAlert', {message: `Pozycja powinna mieć ustaloną ilość gramów na porcję`, triggerManualError:true, type:'positionAddingError'});
          return false;
        }

        // this.selectedProduct[meal.uuid].productQuantity === null || this.selectedProduct[meal.uuid].productQuantity.trim() === '' ?
        //      EventBus.$emit('triggerAlert', {message: `Pozycja powinna mieć ustaloną ilość gramów na porcję`, triggerManualError:true, type:'positionAddingError'})
        //      : null;

        if (this.selectedProduct[meal.uuid].productName == "" || this.selectedProduct[meal.uuid].productQuantity.trim() == "") {

        }
        else {
          let type = this.selectedProduct[meal.uuid].productName.item["@type"].toLowerCase();

          let dataToSend = {
            menuMeal: this.selectedProduct[meal.uuid].productName.meal["@id"], //  śniadanie etc
            type: 1,
            [type]: this.selectedProduct[meal.uuid].productName.item["@id"],
            quantity: parseInt(this.selectedProduct[meal.uuid].productQuantity),
            isActive: true,
            deletedAt: null,
            menu: this.componentData["@id"] // api/menus/123123
          };
          
          this.$Progress.start();
          fetch("/api/menu_positions?context=menuPosition", {
            // create new menu_posiiton and assign it simultaneously to particular menu
            method: "post",
            body: JSON.stringify(dataToSend),
            headers: {
              "Content-Type": "application/json"
            }
          })
            .then(response => {
              (async ()=>{
                  let data = await response.json();
                  EventBus.$emit('triggerAlert', {
                      message: response.ok ? 'Pozycja została dodana' : data, 
                      type:'positionWasAdded'
                  });

                  if(response.ok === true){
                      this.$Progress.finish();
                      copyOfCurrentDay.menuMeals[currentMealIndex].editMode = !copyOfCurrentDay.menuMeals[currentMealIndex].editMode;
                      copyOfCurrentDay.menuMeals[currentMealIndex].menuPositions.push({
                        ...data,
                        material: {
                          ...this.selectedProduct[meal.uuid].productName.item,
                        }
                      });
                  }
                  else {
                      this.$Progress.fail();
                  }
              })()
            })
            .catch(error=>{
              this.$Progress.fail();
              EventBus.$emit('triggerAlert', {message: `Błąd serwera`, triggerManualError:true});
              console.error('Error:', error);
            });
        }


      } 
      else {
        if (this.selectedProduct[meal.uuid].productName == "") {
        } 
        else {
          let type = this.selectedProduct[meal.uuid].productName.item["@type"].toLowerCase();

          let dataToSend = {
            menuMeal: this.selectedProduct[meal.uuid].productName.meal["@id"], //  śniadanie etc
            type: 2,
            [type]: this.selectedProduct[meal.uuid].productName.item["@id"],
            quantity: this.selectedProduct[meal.uuid].productName.item.portionValue,
            isActive: true,
            deletedAt: null,
            menu: this.componentData["@id"] // api/menus/123123
          };
          this.$Progress.start();
          fetch("/api/menu_positions?context=menuPosition", {
            // create new menu_posiiton and assign it simultaneously to particular menu
            method: "post",
            body: JSON.stringify(dataToSend),
            headers: {
              "Content-Type": "application/json"
            }
          })
            .then(response => {
              (async ()=>{
                  let data = await response.json();
                  EventBus.$emit('triggerAlert', {
                      message: response.ok ? 'Pozycja została dodana' : data, 
                      type:'positionWasAdded'
                  });

                  if(response.ok === true){
                      this.$Progress.finish();
                      copyOfCurrentDay.menuMeals[currentMealIndex].editMode = !copyOfCurrentDay.menuMeals[currentMealIndex].editMode;
                      copyOfCurrentDay.menuMeals[currentMealIndex].menuPositions.push({
                        ...data,
                        recipe: {
                          ...this.selectedProduct[meal.uuid].productName.item,
                        }
                      });
                  }
                  else {
                      this.$Progress.fail();
                  }
              })()
            })
            .catch(error=>{
              this.$Progress.fail();
              EventBus.$emit('triggerAlert', {message: `Błąd serwera`, triggerManualError:true});
              console.error('Error:', error);
            });
        }
      }
    },
    deletePosition: function(positionId, index, day, meal) {
      this.$Progress.start();
      fetch("/api/menu_positions/" + positionId + "?context=menuPosition", {
        method: "DELETE",
        headers: {
          "Content-Type": "application/json"
        }
      }).then(response => {
        (async ()=>{
            // let data = await response.json();
            EventBus.$emit('triggerAlert', {
                message: response.ok ? 'Pozycja została usunięta' : data, 
                type:'deletePosition'
            });

            if(response.ok === true){
                this.$Progress.finish();
                let dayIndex = this.componentData.menuDays.findIndex(element=>element.uuid == day.uuid);
                let mealIndex = this.componentData.menuDays[dayIndex].menuMeals.findIndex(element => element.uuid == meal.uuid);
                this.componentData.menuDays[dayIndex].menuMeals[mealIndex].menuPositions = this.componentData.menuDays[dayIndex].menuMeals[mealIndex].menuPositions.filter(element=> element.uuid != positionId);
            }
            else {
                this.$Progress.fail();
            }
        })()
      }).catch(err=>{
        this.$Progress.fail();
        EventBus.$emit('triggerAlert', {message: `Błąd serwera`, triggerManualError:true});
        console.error('Error:', error)
      });
    },
    valuesPerPortion: function(position, type, day, meal) {
      if (position.recipe == null) {
        return ((position.material.nutritionalValuesAssigned[type] * (position.quantity - (position.quantity * position.material.nutritionalValuesAssigned.wastePercent) /100)) / 100).toFixed(2);
      } else {
        return position.recipe.nutritionalValuesAssignedSum[type];
      }
    },
    mealSummPerPortion: function(day, meal, type) {
      let currentMealPositions = meal.menuPositions ? [...meal.menuPositions] : false;
      let summ = {
        [type]: 0
      };
      if(currentMealPositions != false){
        for (let matOrRec of currentMealPositions) {
          if(type == 'price'){
            matOrRec.price !== null ? summ[type] += matOrRec.price : false;
          }
          else {
            if (matOrRec.material !== null) {
              for (let nutValue in matOrRec.material.nutritionalValuesAssigned) {
                if (nutValue == type) {
                  summ[type] +=
                    (matOrRec.material.nutritionalValuesAssigned[nutValue] *
                      (matOrRec.quantity -(matOrRec.quantity * matOrRec.material.nutritionalValuesAssigned.wastePercent) / 100)) / 100;
                }
              }
            } else {
              for (let nutValue in matOrRec.recipe.nutritionalValuesAssignedSum) {
                if (nutValue == type) {
                  summ[type] += matOrRec.recipe.nutritionalValuesAssignedSum[nutValue];
                }
              }
            }
          }
        }
  
        return summ[type].toFixed(2);
      }
      else {
        return summ[type].toFixed(2);
      }
    },
    summForWholeDayPortion(day, type) {
      let currentMeals = day.menuMeals; // [{}]

      let positions = [];

      for(let meal of currentMeals){
        if(meal.menuPositions){
          for(let position of meal.menuPositions){
            positions.push(position)
          }
        }
      }

      let summ = {
        [type]: 0
      };
      for (let matOrRec of positions) {
        if(type == 'price'){
          matOrRec.price !== null ? summ[type] += matOrRec.price : false;
        }
        else {
          if (matOrRec.material !== null) {
            summ[type] +=
              (matOrRec.material.nutritionalValuesAssigned[type] *
                (matOrRec.quantity -
                  (matOrRec.quantity *
                    matOrRec.material.nutritionalValuesAssigned.wastePercent) /
                    100)) /
              100;
          } else {
            summ[type] += matOrRec.recipe.nutritionalValuesAssignedSum[type];
          }
        }
      }
      return summ[type].toFixed(2);
    },
    closeUnsavedPosition(day, meal){
        let copyOfDaysOfWeak = [...this.componentData.menuDays];
        let currentDayIndex = copyOfDaysOfWeak.indexOf(day);
        let currentMealIndex = copyOfDaysOfWeak[currentDayIndex].menuMeals.indexOf(meal);

        let copyOfCurrentDay = {
          ...this.componentData.menuDays[currentDayIndex]
        };
        copyOfCurrentDay.menuMeals[currentMealIndex].editMode = false;

        copyOfDaysOfWeak.splice(currentDayIndex, 1, copyOfCurrentDay);
        this.componentData = {
          ...this.componentData,
          menuDays: [...copyOfDaysOfWeak]
        };

        this.selectedProduct = {
          ...this.selectedProduct,
          [meal.uuid]: {
            productName: "",
            productQuantity: null
          }
        };
    },
    checkIfMealExists(day,mealType){
       let returnValue;
       for(let meal of day.menuMeals){
          if(meal.dMeal.name == mealType.name){
            returnValue = true;
          }
       }
       return returnValue;
    },
    onClickDropdown(event) {
      this.fileName = null;
      this.file = '';
    },
    onFileChanged (event) {
      this.file = event.target.files[0];
      this.fileName = this.file.name;
      if (this.fileName.length > 20) {
        this.fileName = this.fileName.substring(0, 17) + '...';
      }
    },
    async onUpload(meal) {
      const self = this;
      self.imageLoad = true;
      let formData = new FormData();
      formData.append('file', this.file);
      this.$Progress.start();

      let postMedia = await axios.post( '/api/media_objects?context=mediaObject', formData, {
                        headers: {
                          'Content-Type': 'multipart/form-data'
                        }
                      })
                      .then(response => {
                          return response;
                      })
                      .catch(error => {
                        this.$Progress.fail();
                        EventBus.$emit('triggerAlert', {message: error.response.data ? error.response.data : 'Wystąpił nieoczekiwany błąd'});
                        self.imageLoad = false;
                        this.fileName = null;
                        this.file = '';
                      })

      let putMediaAfterPost = await axios.put('/api/menu_meals/'+ meal.uuid +'?context=menuMeal', { image: postMedia.data['@id'] })
                    .then(res => {
                        EventBus.$emit('triggerAlert', {message: 'Plik zapisany poprawnie'});
                        self.imageLoad = false;
                        self.$set(meal, 'image', res.data.image);
                        this.fileName = null;
                        this.file = '';
                        this.$Progress.finish();
                    })
                    .catch(error => {
                      this.$Progress.fail();
                      EventBus.$emit('triggerAlert', {message: error.response.data ? error.response.data : 'Wystąpił nieoczekiwany błąd'});
                      self.imageLoad = false;
                      this.fileName = null;
                      this.file = '';
                    })
    },
    deleteMealImage(meal) {
      const self = this;
      self.imageLoad = true;
      this.$Progress.start();
      axios.put('/api/menu_meals/'+ meal.uuid +'?context=menuMeal', { image: null })
      .then(response => {
        this.$Progress.finish();
        EventBus.$emit('triggerAlert', {message: 'Plik usunięty', type:'deleteMealImage'});
        self.$set(meal, 'image', null);
        self.imageLoad = false;
      })
      .catch(error => {
        this.$Progress.fail();
        EventBus.$emit('triggerAlert', {message: `Nie udało się usunąć pliku`, triggerManualError:true});
        self.imageLoad = false;
      })
    },
    checkMealsPositions(meals){
      if(meals.length>0){
        return meals.some(meal=>{
          return meal.menuPositions.length>0
        })
      }
    },
    countPrice(meal){
      if(this.selectedProduct[meal.uuid].productName.item.inventories[0].price == null){
        return 'Brak';
      }
      else {
        if(this.selectedProduct[meal.uuid].productQuantity == '' || this.selectedProduct[meal.uuid].productQuantity == null){
          return (parseFloat(this.selectedProduct[meal.uuid].productName.item.inventories[0].price)).toFixed(2)
        }
        else {
          return ((parseFloat(this.selectedProduct[meal.uuid].productName.item.inventories[0].price)).toFixed(2) * parseFloat(this.selectedProduct[meal.uuid].productQuantity)).toFixed(2)
        }
      }
    }
  },
  watch: {
    currentSelectedMeal() {
      if (this.currentSelectedMeal !== "") {
        
        let currentDay = this.componentData.menuDays.find(element => {
          return (
            element.dayOfWeek.name == this.currentSelectedMeal.selectedDay.dayOfWeek.name
          );
        });

        if (currentDay != false && currentDay.menuMeals.some(element =>element.dMeal.uuid == this.currentSelectedMeal.typeOfMeal.uuid) == false) {
          let dataToSend = {
            menu: "/api/menus/" + this.menuid,
            dMeal: this.currentSelectedMeal.typeOfMeal["@id"],
            menuDay: this.currentSelectedMeal.selectedDay["@id"],
            isActive: true
          };
          this.$Progress.start();
          this.isUpdated = true;
          fetch("/api/menu_meals?context=menuMeal", {
            headers: {
              "content-type": "application/json"
            },
            method: "POST",
            body: JSON.stringify(dataToSend)
          }).then(response => {
             return response.json();
          }).then(data=>{
            this.$Progress.finish();
            if (data) {
              let selectedTypeOfMeal = this.typeOfMeal.find(element => {
                return this.currentSelectedMeal.typeOfMeal.name == element.name;
              });
              currentDay.menuMeals.push({
                ...data,
                dMeal: {
                  ...this.currentSelectedMeal.typeOfMeal,
                },
                editMode:false
              });

              currentDay.menuMeals.sort((a, b) => {
                return a.dMeal.dOrder - b.dMeal.dOrder;
              });
              this.currentSelectedMeal = "";
            }
            this.isUpdated = false;
          });
        }
      }
    },
    currentStatus(newValue, oldValue){
      EventBus.$emit('passMenuStatus', this.currentStatus);
    }
  }
};
</script>


<style lang="scss" scoped>
@import '../../../css/variables';
.imageNameCentered {
    margin: 0 auto;
}
.vdp-datepicker__calendar {
  left: -120px;
  top: 30px;
}
.nav-link.active:hover {
  color: #8796a5 !important;
  background-color: #fff !important;
}
.vdp-datepicker {
  input {
    background-color: transparent;
    border: 0.5px solid;
    border-color: #707070;
    color: #9f9c9c;
    text-align: center;
  }
}
.meal-select {
  color: $thead-txt-dark-color;
  border-radius: 5px;
  border: 1px solid $form-input-border-col;
  padding: 3px 0 3px 10px;
  &--disabled {
    cursor:not-allowed;
    // background-color: lighten(#66d6ff,10%);
    // // background-color: #c4c4c4;
    // color: black;
    // font-weight: bold;
    // box-shadow: 0px 5px darken(#66d6ff,15%) !important;
  }
}
.meal-select__option {
  font-weight: 600;
}
.meal-select__option[disabled="disabled"] {
   color:$tbody-txt-col;
}
.media {
  display: flex;
  align-items: default !important;
}
.media-body {
  display: flex;
  align-items: center;
  img {
    width: 13px;
    height: auto;
    margin-left: 5px;
    cursor: pointer;
  }
}

.table--menu {
  overflow: auto !important;
  white-space: nowrap;
  th:nth-child(2){
    width:210px;
  }
  th:last-child {
    width: 60px;
  }
}
.menu-action {
  text-align:center;
}
// dropdown hover action
.dropdown-menu-gallery-button {
  position: relative;
  .dropdown-menu-zoom {
    display: none;
    color: $white;
    text-shadow: 0 0 3px rgba(0,0,0,.5);
    position: absolute;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    -ms-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }
  &:hover {
    .dropdown-menu-zoom {
      display: block;
    }
  }
  .dropdown-menu-camera {
    position: absolute;
    padding:4px;
    border:1px solid #ccc;
    border-radius: 50%;
    top:0;
    left:0;
  }
}

</style>


