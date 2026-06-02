<script setup lang="ts">
import {
    Card,
    CardContent,
    CardAction,
    CardHeader,
    CardTitle,
} from '@/components/ui/card';
import { IconCar, IconGasStation, IconPlus, IconX, IconMedal2 } from '@tabler/icons-vue';
import { Button } from '@/components/ui/button';
import { Label } from '@/components/ui/label';
import type { CarouselApi } from '@/components/ui/carousel'
import {
    Carousel,
    CarouselContent,
    CarouselItem,
} from '@/components/ui/carousel'
import { ref, computed, watch } from 'vue'
import { InputGroup, InputGroupAddon, InputGroupText, InputGroupInput } from '@/components/ui/input-group';
import { useStorage, watchOnce } from '@vueuse/core';
import { Input } from '@/components/ui/input';
import { Switch } from '@/components/ui/switch';
import {
    Table,
    TableBody,
    TableCaption,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
} from '@/components/ui/table'
type CarInfo = { id: string; name: string; fuelConsumption: number; refuelAmt: number }
const carInfoList = useStorage<CarInfo[]>('carInfo', [{ id: 'car-1', name: "Your car", fuelConsumption: 5.5, refuelAmt: 20 }]);

const api = ref<CarouselApi>()
const totalCount = ref(0)
const current = ref(0)

type Station = { id: string; name: string; price: number; distance: number; consumption?: number; overrideConsumption: boolean }
const stationList = ref<Station[]>([{ id: "Ex1", name: "Example station", price: 3.12, distance: 15, overrideConsumption: false }]);



function setApi(val: CarouselApi) {
    api.value = val
}
function goToSlide(index: number) {
    api.value?.scrollTo(index)
    current.value = index
}

function addCar() {
    carInfoList.value.push({
        id: crypto.randomUUID?.() ?? Math.random().toString(36).slice(2),
        name: "New Car",
        fuelConsumption: 3,
        refuelAmt: 20,
    })
}

function addStation() {
    stationList.value.push({
        id: crypto.randomUUID?.() ?? Math.random().toString(36).slice(2),
        name: "New station",
        distance: 5,
        price: 3.5,
        overrideConsumption: false
    })
}

function removeStation(index: number) {
    stationList.value.splice(index, 1)
}

function removeCar(index: number) {
    carInfoList.value.splice(index, 1)
    api.value?.scrollPrev(true)
}
watchOnce(api, (api) => {
    if (!api)
        return

    totalCount.value = api.scrollSnapList().length
    current.value = api.selectedScrollSnap()
    api.on('select', () => {
        current.value = api.selectedScrollSnap()
    })
})

watch(current, (newVal) => {
  console.log(`${newVal} selected`)
  const station = stationList.value[0]
  const car = carInfoList.value[current.value]
  console.log(`Distance: ${station.distance}, consumption: ${car.fuelConsumption}, refuel: ${car.refuelAmt}, price: ${station.price}`)
  const val = ((station.distance * car.fuelConsumption)/100 + car.refuelAmt) * station.price
  console.log(`Litre used to cover distance: ${(station.distance * car.fuelConsumption)/100}`)
  console.log(`Total litre used: ${(station.distance * car.fuelConsumption)/100 + car.refuelAmt}`)
  console.log(typeof station.distance, typeof carInfoList.value[current.value].refuelAmt)
  console.log(`Cost: ${val}`)
})

const rankedStationList = computed(() => {
    const costCalStationList = stationList.value.map(station => ({...station, 
        cost: ((station.distance * carInfoList.value[current.value].fuelConsumption)/100 + carInfoList.value[current.value].refuelAmt) * station.price }))
    return costCalStationList.sort((a, b) => a.cost - b.cost)
})
</script>

<template>
    <div class="flex flex-col gap-4 p-3">
        <Carousel class="w-full max-w-m" @init-api="setApi" :opts="{
            align: 'start',
        }">
            <CarouselContent>
                <CarouselItem v-for="(carInfo, index) in carInfoList" :key="carInfo.id">
                    <div class="py-1">
                        <Card>
                            <CardHeader>
                                <CardTitle class="flex items-center gap-2">
                                    <IconCar stroke-width="1.5" class="text-green-500" />
                                    <input type="text" v-model="carInfo.name" />
                                </CardTitle>
                                <CardAction>
                                    <Button variant="ghost" @click="removeCar(index)">
                                        <IconX />
                                    </Button>
                                </CardAction>
                            </CardHeader>
                            <CardContent>
                                <div class="flex flex-col w-full gap-4 md:flex-row">
                                    <div class="text-muted-foreground flex flex-col gap-2">
                                        <Label for="carname">Consumption</Label>
                                        <InputGroup class="w-48 md:w-72">
                                            <InputGroupInput placeholder="0.00" v-model.number="carInfo.fuelConsumption" />
                                            <InputGroupAddon align="inline-end">
                                                <InputGroupText>L/100km</InputGroupText>
                                            </InputGroupAddon>
                                        </InputGroup>
                                    </div>
                                    <div class="text-muted-foreground flex flex-col gap-2">
                                        <Label for="carname">Refuel amount</Label>
                                        <InputGroup class="w-48 md:w-72">
                                            <InputGroupInput placeholder="0.00" v-model.number="carInfo.refuelAmt" />
                                            <InputGroupAddon align="inline-end">
                                                <InputGroupText>L</InputGroupText>
                                            </InputGroupAddon>
                                        </InputGroup>
                                    </div>
                                </div>
                            </CardContent>
                        </Card>
                    </div>
                </CarouselItem>
            </CarouselContent>
        </Carousel>
        <!-- Carousel navigation -->
        <div class="flex justify-center gap-4 mb-3">
            <Button v-for="(c, index) in carInfoList" :key="c.id" size="icon-xs"
                class="rounded-full transition-colors duration-200"
                :class="current === index ? 'bg-slate-900 dark:bg-slate-100' : 'bg-slate-300 dark:bg-slate-700 hover:bg-slate-400'"
                :aria-current="current === index ? 'true' : undefined" :aria-label="`Go to ${c.name}`"
                @click="goToSlide(index)" />
            <Button size="icon-xs" class="rounded-full" aria-label="Submit" @click="addCar">
                <IconPlus stroke-width="2" />
            </button>
        </div>
        <!-- Stations input -->
        <Card>
            <CardHeader>
                <CardTitle class="flex items-center gap-2">
                    <IconGasStation stroke-width="1.5" class="text-green-500" />Stations
                </CardTitle>
                <CardAction>
                    <Button @click="addStation()"><IconPlus /></Button>
                </CardAction>
            </CardHeader>
            <CardContent>
                <div class="flex flex-col gap-2">
                    <Card v-for="(station, index) in stationList" :key="station.id">
                        <CardHeader>
                            <CardTitle>
                                <div class="text-muted-foreground flex flex-col gap-2">
                                    <Input class="w-48 md:w-72" placeholder="Station name" v-model="station.name" />
                                </div>
                            </CardTitle>
                            <CardAction>
                                <Button variant="ghost" @click="removeStation(index)">
                                    <IconX />
                                </Button>
                            </CardAction>
                        </CardHeader>
                        <CardContent>
                            <div class="flex flex-col w-full gap-4 md:flex-row">
                                <div class="text-muted-foreground flex flex-col gap-2">
                                    <Label for="price">Price</Label>
                                    <InputGroup class="w-48 md:w-72">
                                        <InputGroupAddon align="inline-start">
                                            <InputGroupText>$</InputGroupText>
                                        </InputGroupAddon>
                                        <InputGroupInput placeholder="0.00" v-model.number="station.price" />
                                        <InputGroupAddon align="inline-end">
                                            <InputGroupText>/L</InputGroupText>
                                        </InputGroupAddon>
                                    </InputGroup>
                                </div>
                                <div class="text-muted-foreground flex flex-col gap-2">
                                    <Label for="price">Distance</Label>
                                    <InputGroup class="w-48 md:w-72">
                                        <InputGroupInput placeholder="0.00" v-model.number="station.distance" />
                                        <InputGroupAddon align="inline-end">
                                            <InputGroupText>km</InputGroupText>
                                        </InputGroupAddon>
                                    </InputGroup>
                                </div>
                                <div class="flex items-center space-x-2">
                                    <Label for="override-cons">Override Fuel Consumption</Label>
                                    <Switch name="override-cons" v-model="station.overrideConsumption" />
                                </div>
                                <div v-if="station.overrideConsumption"
                                    class="text-muted-foreground flex flex-col gap-2">
                                    <Label for="carname">Consumption</Label>
                                    <InputGroup class="w-48 md:w-72">
                                        <InputGroupInput placeholder="0.00" v-model.number="station.consumption" />
                                        <InputGroupAddon align="inline-end">
                                            <InputGroupText>L/100km</InputGroupText>
                                        </InputGroupAddon>
                                    </InputGroup>
                                </div>
                            </div>
                        </CardContent>
                    </Card>
                </div>
            </CardContent>
        </Card>

        <Card>
            <CardHeader>
                <CardTitle class="flex items-center gap-2">
                    <IconMedal2 stroke-width="1.5" class="text-green-500" /> Results
                </CardTitle>
            </CardHeader>
            <CardContent>
                <Table>
                    <TableCaption>Stations ranking</TableCaption>
                    <TableHeader>
                        <TableRow>
                            <TableHead>
                                Location
                            </TableHead>
                            <TableHead>Price</TableHead>
                            <TableHead>Distance</TableHead>
                            <TableHead class="text-right">
                                Cost
                            </TableHead>
                        </TableRow>
                    </TableHeader>
                    <TableBody>
                        <TableRow v-for="(st, i) in rankedStationList" :key="st.id">
                            <TableCell class="font-medium">
                                {{ st.name }}
                            </TableCell>
                            <TableCell>${{ st.price }}/L</TableCell>
                            <TableCell>{{ st.distance }} km</TableCell>
                            <TableCell class="flex flex-row-reverse items-center">
                                <span>
                                    ${{ st.cost.toFixed(2) }}
                                </span>
                                <IconMedal2 class="text-red-500 w-4" v-if="i==0"/>
                            </TableCell>
                        </TableRow>
                    </TableBody>
                </Table>
            </CardContent>
        </Card>
    </div>
</template>
