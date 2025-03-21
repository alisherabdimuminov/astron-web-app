<script setup lang="ts">
import { LucideChevronLeft, LucideChevronRight } from 'lucide-vue-next';


interface IVariant {
    variant: string
    var_id: string
    is_right: string
}

interface ITest {
    savol_id: string
    savol_nomi: string
    test_file: string
    variant: IVariant[]
    is_selected: boolean
    is_solved: boolean
    selected: string
}


const route = useRoute();
const router = useRouter();

const userStore = useUserStore();
const subjectsStore = useSubjectsStore();

const { token } = storeToRefs(userStore);

const isLoading = ref(true);
const tests = ref<ITest[]>([]);



const getTests = async () => {
    isLoading.value = true;
    let response = await $fetch<string>("https://astrontest.uz/mobile-api/api/uz/testlaruz?lang=uz", {
        method: "POST",
        body: JSON.stringify({
            "token": token.value,
            "subjectid": route.params.subjectid,
            "classid": route.params.classid,
            "themeid": route.params.themeid,
        }),
        headers: {
            "Content-Type": "application/json",
        }
    });

    console.log(response);

    tests.value = JSON.parse(response);
    tests.value = tests.value.map((test) => ({ ...test, is_solved: false, selected: "", is_selected: false }))
    isLoading.value = true;
}

const selected = computed(() => {
    return tests.value.filter(test => test.is_selected).length === tests.value.length;
});

const calculate = computed(() => () => {
    let all = tests.value.length;
    let correct = tests.value.filter(test => test.is_solved).length;
    let incorrect = all - correct;
    return { all, correct, incorrect };
});


definePageMeta({
    middleware: ["is-telegram", "get-subjects"],
});

onMounted(() => {
    getTests();
    isLoading.value = false;
});
</script>

<template>
    <div class="h-screen w-full">
        <div class="bg-background flex items-center justify-between gap-2 h-[3rem] p-2 border-b">
            <div class="flex items-center gap-1">
                <div class="border rounded-full p-1" @click="router.back()">
                    <LucideChevronLeft />
                </div>
                <p>Test</p>
            </div>
            <div>
                <Drawer v-if="selected">
                    <DrawerTrigger as-child>
                        <Button size="sm">Natija</Button>
                    </DrawerTrigger>
                    <DrawerContent class="h-3/4 p-5">
                        <DrawerHeader>
                            <DrawerTitle>Natija</DrawerTitle>
                            <DrawerDescription></DrawerDescription>
                        </DrawerHeader>
                        <div class="border overflow-auto rounded-md">
                            <Table>
                                <TableBody>
                                    <TableRow>
                                        <TableCell class="border-r">Savollar soni</TableCell>
                                        <TableCell>{{ calculate().all }}</TableCell>
                                    </TableRow>
                                    <TableRow>
                                        <TableCell class="border-r">To'g'ri javob</TableCell>
                                        <TableCell>{{ calculate().correct }}</TableCell>
                                    </TableRow>
                                    <TableRow>
                                        <TableCell class="border-r">Noto'g'ri javob</TableCell>
                                        <TableCell>{{ calculate().incorrect }}</TableCell>
                                    </TableRow>
                                    <TableRow>
                                        <TableCell class="border-r">Foiz</TableCell>
                                        <TableCell>{{ Math.ceil((calculate().correct / calculate().all) * 100) }}%</TableCell>
                                    </TableRow>
                                </TableBody>
                            </Table>
                        </div>
                    </DrawerContent>
                </Drawer>
            </div>
        </div>
        <div class="h-[calc(100%-3rem)] flex flex-col gap-2 px-5">
            <ScrollArea class="h-full">
                <br>
                <div class="flex flex-col gap-3">
                    <div v-for="test, index in tests" class="bg-accent/30 rounded-md p-2">
                        <p class="text-lg">{{ index+1 }}. {{ test.savol_nomi }}</p>
                        <div class="flex items-center justify-center">
                            <Dialog>
                                <DialogTrigger as-child>
                                    <img class="h-32" v-if="test.test_file && test.test_file !== 'empty'" :src="`https://astrontest.uz/mypage/uploads/${test.test_file}`" alt="">
                                </DialogTrigger>
                                <DialogContent class="p-0">
                                    <img class="w-full h-full" v-if="test.test_file && test.test_file !== 'empty'" :src="`https://astrontest.uz/mypage/uploads/${test.test_file}`" alt="">
                                </DialogContent>
                            </Dialog>
                        </div>
                        <RadioGroup :name="`answer_${index+1}`" v-model="test.selected" @update:model-value="() => { test.is_selected = true; test.is_solved = (test.variant.filter(variant => variant.var_id === test.selected)[0].is_right === '1' && test.variant.filter(variant => variant.var_id === test.selected)[0].var_id === test.selected) }" :disabled="test.is_selected">
                            <div v-for="variant in test.variant" class="flex items-center space-x-2">
                                <RadioGroupItem :value="variant.var_id" :id="`answer_${index+1}_${variant.var_id}`" />
                                <Label :for="`answer_${index+1}_${variant.var_id}`"><span v-if="variant.is_right === '1' && test.selected">✅</span> <span v-if="test.selected === variant.var_id && variant.is_right !== '1'">❌</span> {{ variant.variant }}</Label>
                            </div>
                        </RadioGroup>
                    </div>
                </div>
                <br>
            </ScrollArea>
        </div>
    </div>
</template>