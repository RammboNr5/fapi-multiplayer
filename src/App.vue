<script setup>
import {computed, ref, watch, onMounted} from 'vue'

const BASEDAMAGE = 'basedamage'
const CONSECUTIVE = 'consecutive'
const XP = 'xp'
const GP = 'gp'
const CHARGE = 'charge'
const PERCENTDAMAGE = 'percentdamage'
const CRIT = 'crit'

const bosstier = ref(1)
const playerlevel = ref(0)
const currentGp = ref(0)
const currentXp = ref(0)
const critCounter = ref(0)

const basedamage = ref(0)
const basedamageCost = ref(5)
const damageAfterBaseUpgrade = ref(212)
const basedamageEfficiency = ref(0.2083)

const percentdamage = ref(0)
const percentdamageCost = ref(20)
const damageAfterPercentdamageUpgrade = ref(216)
const percentdamageEfficiency = ref(0.893)

const consecutive = ref(0)
const consecutiveCost = ref(25)
const damageAfterConsecutiveUpgrade = ref(216)
const consecutiveEfficiency = ref(0.893)

const xp = ref(0)
const xpCost = ref(5)

const gp = ref(0)
const gpCost = ref(5)

const charge = ref(0)
const chargeCost = ref(10)
const damageAfterChargeUpgrade = ref(216)
const chargeEfficiency = ref(0.893)

const crit = ref(0)
const critCost = ref(25)
const damageAfterCritUpgrade = ref(216)
const critEfficiency = ref(0.893)

// Desired upgrade values
const desiredBasedamage = ref(0)
const desiredPercentageDamage = ref(0)
const desiredConsecutive = ref(0)
const desiredXp = ref(0)
const desiredGp = ref(0)
const desiredCharge = ref(0)
const desiredCrit = ref(0)

// Second set of desired upgrade values
const desired2Basedamage = ref(0)
const desired2PercentageDamage = ref(0)
const desired2Consecutive = ref(0)
const desired2Xp = ref(0)
const desired2Gp = ref(0)
const desired2Charge = ref(0)
const desired2Crit = ref(0)

const buffDamage = ref(0)
const buffXp = ref(0)
const buffGp = ref(0)

const totalDamage = ref(20 + 21 + 22 + 23 + 24 + 25 + 26 + 27)
const rewardXp = ref(12)
const rewardGp = ref(12)
const levelAfterBoss = ref("")
const levelAfterBossDesired = ref("")
const levelAfterBossDesired2 = ref("")

const rank = ref("0")

onMounted(() => {
    updateEverything()
})

// Formatted efficiency values for display
const formattedBasedamageEfficiency = computed(() => {
    return basedamageEfficiency.value.toFixed(3)
})

const formattedPercentageDamageEfficiency = computed(() => {
    return percentdamageEfficiency.value.toFixed(3)
})

const formattedConsecutiveEfficiency = computed(() => {
    return consecutiveEfficiency.value.toFixed(3)
})

const formattedChargeEfficiency = computed(() => {
    return chargeEfficiency.value.toFixed(3)
})

const formattedCritEfficiency = computed(() => {
    return critEfficiency.value.toFixed(3)
})

// Add watchers to copy values from base inputs to desired inputs
watch(basedamage, (newVal) => {
    desiredBasedamage.value = newVal
    desired2Basedamage.value = newVal
})

watch(percentdamage, (newVal) => {
    desiredPercentageDamage.value = newVal
    desired2PercentageDamage.value = newVal
})

watch(crit, (newVal) => {
    desiredCrit.value = newVal
    desired2Crit.value = newVal
})

watch(consecutive, (newVal) => {
    desiredConsecutive.value = newVal
    desired2Consecutive.value = newVal
})

watch(xp, (newVal) => {
    desiredXp.value = newVal
    desired2Xp.value = newVal
})

watch(gp, (newVal) => {
    desiredGp.value = newVal
    desired2Gp.value = newVal
})

watch(charge, (newVal) => {
    desiredCharge.value = newVal
    desired2Charge.value = newVal
})

// Total cost to reach desired upgrades
const totalUpgradeCost = computed(() =>
    calculateTotalUpgradeCost(
        basedamage.value, desiredBasedamage.value,
        consecutive.value, desiredConsecutive.value,
        xp.value, desiredXp.value,
        gp.value, desiredGp.value,
        charge.value, desiredCharge.value,
        percentdamage.value, desiredPercentageDamage.value,
        crit.value, desiredCrit.value
    )
)

// Total cost to reach desired upgrades - second set
const totalUpgradeCost2 = computed(() =>
    calculateTotalUpgradeCost(
        basedamage.value, desired2Basedamage.value,
        consecutive.value, desired2Consecutive.value,
        xp.value, desired2Xp.value,
        gp.value, desired2Gp.value,
        charge.value, desired2Charge.value,
        percentdamage.value, desired2PercentageDamage.value,
        crit.value, desired2Crit.value
    )
)

// Calculate total damage with desired upgrades
const totalDesiredDamage = computed(() => {
    let desiredCharges = 8 + desiredCharge.value + Math.floor(playerlevel.value / 25);
    return calcDamage(desiredCharges, desiredBasedamage.value, desiredConsecutive.value, desiredPercentageDamage.value, desiredCrit.value, critCounter.value, playerlevel.value);
})

// Calculate total damage with desired upgrades - second set
const totalDesiredDamage2 = computed(() => {
    let desiredCharges = 8 + desired2Charge.value + Math.floor(playerlevel.value / 25);
    return calcDamage(desiredCharges, desired2Basedamage.value, desired2Consecutive.value, desired2PercentageDamage.value, desired2Crit.value, critCounter.value, playerlevel.value);
})

// Calculate XP gain for target 1
const desiredRewardXp = computed(() => {
    return calcRewardXpValue(desiredXp.value);
})

// Calculate XP gain for target 2
const desired2RewardXp = computed(() => {
    return calcRewardXpValue(desired2Xp.value);
})

// Calculate GP gain for target 1
const desiredRewardGp = computed(() => {
    return calcRewardGpValue(desiredGp.value);
})

// Calculate GP gain for target 2
const desired2RewardGp = computed(() => {
    return calcRewardGpValue(desired2Gp.value);
})

// Calculate damage increase percentage
const damageIncreasePercentage = computed(() => {
    if (totalDamage.value === 0) return 0;
    return Math.round(((totalDesiredDamage.value - totalDamage.value) / totalDamage.value) * 100);
})

// Calculate damage increase percentage - second set
const damageIncreasePercentage2 = computed(() => {
    if (totalDamage.value === 0) return 0;
    return Math.round(((totalDesiredDamage2.value - totalDamage.value) / totalDamage.value) * 100);
})

// Calculate XP increase percentage
const xpIncreasePercentage = computed(() => {
    if (rewardXp.value === 0) return 0;
    return Math.round(((desiredRewardXp.value - rewardXp.value) / rewardXp.value) * 100);
})

// Calculate XP increase percentage - second set
const xpIncreasePercentage2 = computed(() => {
    if (rewardXp.value === 0) return 0;
    return Math.round(((desired2RewardXp.value - rewardXp.value) / rewardXp.value) * 100);
})

// Calculate GP increase percentage
const gpIncreasePercentage = computed(() => {
    if (rewardGp.value === 0) return 0;
    return Math.round(((desiredRewardGp.value - rewardGp.value) / rewardGp.value) * 100);
})

// Calculate GP increase percentage - second set
const gpIncreasePercentage2 = computed(() => {
    if (rewardGp.value === 0) return 0;
    return Math.round(((desired2RewardGp.value - rewardGp.value) / rewardGp.value) * 100);
})

const highestDamage = computed(() => {
    const damages = [damageAfterBaseUpgrade.value, damageAfterConsecutiveUpgrade.value, damageAfterChargeUpgrade.value, damageAfterPercentdamageUpgrade.value, damageAfterCritUpgrade.value];
    return Math.max(...damages);
})

const bestEfficiency = computed(() => {
    const efficiencies = [basedamageEfficiency.value, consecutiveEfficiency.value, chargeEfficiency.value, percentdamageEfficiency.value, critEfficiency.value];
    return Math.min(...efficiencies.filter(entry => entry > 0));
})

function updateEverything() {
    calcTotalDamage()
    calcRewardXp()
    calcRewardGp()
    updateCosts()
    calcEfficiency()
    updateLevel()
}

function calcTotalDamage() {
    let charges = 8 + charge.value + Math.floor(playerlevel.value / 25);
    totalDamage.value = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
}

function calcEfficiency() {
    calcBasedamageEfficiency()
    calcConsecutiveEfficiency()
    calcChargeEfficiency()
    calcPercentdamageEfficiency()
    calcCritEfficiency()
}

function expUntilNextLevel(level) {
    return Math.floor((10 + (level * 5)) * Math.pow(1.02, level))
}

function calcBasedamageEfficiency() {
    let charges = 8 + charge.value + Math.floor(playerlevel.value / 25);
    let currentDamage = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let damageAfterUpgrade = calcDamage(charges, basedamage.value + 1, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let cost = getUpgradeCost(BASEDAMAGE, basedamage.value)
    damageAfterBaseUpgrade.value = damageAfterUpgrade
    basedamageEfficiency.value = cost / (damageAfterUpgrade - currentDamage)
}

function calcConsecutiveEfficiency() {
    let charges = 8 + charge.value + Math.floor(playerlevel.value / 25);
    let currentDamage = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let damageAfterUpgrade = calcDamage(charges, basedamage.value, consecutive.value + 1, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let cost = getUpgradeCost(CONSECUTIVE, consecutive.value)
    damageAfterConsecutiveUpgrade.value = damageAfterUpgrade
    consecutiveEfficiency.value = cost / (damageAfterUpgrade - currentDamage)
}

function calcChargeEfficiency() {
    let charges = 8 + charge.value + Math.floor(playerlevel.value / 25);
    let currentDamage = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let damageAfterUpgrade = calcDamage(charges + 1, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let cost = getUpgradeCost(CHARGE, charge.value)
    damageAfterChargeUpgrade.value = damageAfterUpgrade
    chargeEfficiency.value = cost / (damageAfterUpgrade - currentDamage)
}

function calcPercentdamageEfficiency() {
    let charges = 8 + charge.value + Math.floor(playerlevel.value / 25);
    let currentDamage = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let damageAfterUpgrade = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value + 1, crit.value, critCounter.value, playerlevel.value)
    let cost = getUpgradeCost(CHARGE, charge.value)
    damageAfterChargeUpgrade.value = damageAfterUpgrade
    percentdamageEfficiency.value = cost / (damageAfterUpgrade - currentDamage)
}

function calcCritEfficiency() {
    let charges = 8 + charge.value + Math.floor(playerlevel.value / 25);
    let currentDamage = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value, critCounter.value, playerlevel.value)
    let damageAfterUpgrade = calcDamage(charges, basedamage.value, consecutive.value, percentdamage.value, crit.value + 1, critCounter.value - 1, playerlevel.value)
    let cost = getUpgradeCost(CRIT, charge.value)
    console.log(charges, currentDamage, damageAfterUpgrade, cost)
    damageAfterChargeUpgrade.value = damageAfterUpgrade
    critEfficiency.value = cost / (damageAfterUpgrade - currentDamage)
}

function updateCosts() {
    basedamageCost.value = getUpgradeCost(BASEDAMAGE, basedamage.value)
    consecutiveCost.value = getUpgradeCost(CONSECUTIVE, consecutive.value)
    xpCost.value = getUpgradeCost(XP, xp.value)
    gpCost.value = getUpgradeCost(GP, gp.value)
    chargeCost.value = getUpgradeCost(CHARGE, charge.value)
    percentdamageCost.value = getUpgradeCost(PERCENTDAMAGE, percentdamage.value)
    critCost.value = getUpgradeCost(CRIT, crit.value)
}

function updateLevel() {
    levelAfterBoss.value = calcLevelAfterBoss(rewardXp.value)
    levelAfterBossDesired.value = calcLevelAfterBoss(desiredRewardXp.value)
    levelAfterBossDesired2.value = calcLevelAfterBoss(desired2RewardXp.value)
}

function calcLevelAfterBoss(gainedXp) {
    let level = playerlevel.value
    let requiredXp = expUntilNextLevel(level)
    let actualXp = currentXp.value + gainedXp

    if (actualXp < requiredXp) {
        return level + " (" + actualXp + "xp | " + (actualXp / requiredXp * 100).toFixed(2) + "%)"
    }

    do {
        level++
        actualXp -= requiredXp
        requiredXp = expUntilNextLevel(level)
    } while (actualXp >= requiredXp)

    return level + " (" + actualXp + "xp | " + (actualXp / requiredXp * 100).toFixed(2) + "%)"
}

function getUpgradeCost(type, level) {
    switch (type) {
        case BASEDAMAGE:
            return Math.floor((5 + level) * (1 + level * 0.1))
        case CONSECUTIVE:
            return Math.floor((20 + 25 * level) * Math.pow(1.2, level))
        case XP:
        case GP:
            return Math.floor((5 + level) * (1 + level * 0.05))
        case CHARGE:
            return Math.floor(10 * Math.pow(2.0, level))
        case PERCENTDAMAGE:
            return Math.floor((20 + 25 * level) * Math.pow(1.4, level))
        case CRIT:
            return Math.floor((25 + 25 * level) * Math.pow(2.0, level))
        default:
            console.error('Unknown upgrade type: ', type)
            return 0
    }
}

function calculateUpgradeCost(type, startLevel, endLevel) {
    if (endLevel <= startLevel) return 0

    let totalCost = 0
    for (let level = startLevel; level < endLevel; level++) {
        totalCost += getUpgradeCost(type, level)
    }
    return totalCost
}

function calculateTotalUpgradeCost(
    baseStart, baseEnd,
    consStart, consEnd,
    xpStart, xpEnd,
    gpStart, gpEnd,
    chargeStart, chargeEnd,
    percentdamageStart, percentdamageEnd,
    critStart, critEnd
) {
    return calculateUpgradeCost(BASEDAMAGE, baseStart, baseEnd)
        + calculateUpgradeCost(CONSECUTIVE, consStart, consEnd)
        + calculateUpgradeCost(XP, xpStart, xpEnd)
        + calculateUpgradeCost(GP, gpStart, gpEnd)
        + calculateUpgradeCost(CHARGE, chargeStart, chargeEnd)
        + calculateUpgradeCost(PERCENTDAMAGE, percentdamageStart, percentdamageEnd)
        + calculateUpgradeCost(CRIT, critStart, critEnd)
}

function calcDamage(charges, basedamageUpgrades, consecutiveUpgrades, percentDamageUpgrades, critUpgrades, critcounter, level) {
    let damage = 0
    let tempCritCounter = critcounter
    for (let hit = 0; hit < charges; hit++) {
        if (critUpgrades > 0) {
            --tempCritCounter
        }
        let newDamage = 20 + level
        newDamage += basedamageUpgrades * 3
        newDamage += hit * (1 + consecutiveUpgrades)
        newDamage *= (1 + buffDamage.value / 100)
        newDamage *= (1 + 0.01 * level)
        newDamage *= Math.pow(1.05, percentDamageUpgrades)
        if (critUpgrades > 0 && tempCritCounter <= 0) {
            newDamage *= 5.0
            tempCritCounter = 15 - critUpgrades
            console.log("crit")
        }
        damage += Math.ceil(newDamage)
    }
    return Math.ceil(damage)
}

function calcRewardXp() {
    rewardXp.value = calcRewardXpValue(xp.value)
}

function calcRewardXpValue(xpLevel) {
    let reward = 10 + bosstier.value
    reward *= (1 + xpLevel * 0.05) * Math.pow(1.05, bosstier.value)
    reward *= (1 + rank.value / 100)
    reward *= (1 + buffXp.value / 100)
    return Math.ceil(reward)
}

function calcRewardGp() {
    rewardGp.value = calcRewardGpValue(gp.value)
}

function calcRewardGpValue(gpLevel) {
    let reward = 10 + bosstier.value
    reward *= (1 + gpLevel * 0.05) * Math.pow(1.05, bosstier.value)
    reward *= (1 + rank.value / 100)
    reward *= (1 + buffGp.value / 100)
    return Math.ceil(reward)
}

function optimizeUpgrades() {
    // Make sure the desired2 values match current values at the start
    desired2Basedamage.value = basedamage.value;
    desired2Consecutive.value = consecutive.value;
    desired2Charge.value = charge.value;
    desired2Xp.value = xp.value;
    desired2Gp.value = gp.value;
    desired2PercentageDamage.value = percentdamage.value;
    desired2Crit.value = crit.value;

    // GP available for upgrades
    const availableGp = currentGp.value;
    if (availableGp <= 0) return; // No GP to spend

    // Current best configuration
    let bestDamage = calcDamage(
        8 + charge.value + Math.floor(playerlevel.value / 25),
        basedamage.value,
        consecutive.value,
        percentdamage.value,
        crit.value,
        critCounter.value,
        playerlevel.value
    );

    // Try different combinations of upgrades
    let bestBase = basedamage.value;
    let bestConsecutive = consecutive.value;
    let bestCharge = charge.value;
    let bestPercentDamage = percentdamage.value;
    let bestCrit = crit.value;

    // Start with current values
    let baseStart = basedamage.value;
    let consStart = consecutive.value;
    let chargeStart = charge.value;
    let percentDamageStart = percentdamage.value;
    let critStart = crit.value;

    // Set reasonable limits for iteration
    // Calculate how many levels we could potentially buy based on GP
    let maxBaseLevels = 0;
    let tempCost = 0;
    let tempLevel = baseStart;
    while (tempCost < availableGp) {
        tempCost += getUpgradeCost(BASEDAMAGE, tempLevel);
        if (tempCost <= availableGp) {
            maxBaseLevels++;
            tempLevel++;
        } else {
            break;
        }
    }

    let maxConsLevels = 0;
    tempCost = 0;
    tempLevel = consStart;
    while (tempCost < availableGp) {
        tempCost += getUpgradeCost(CONSECUTIVE, tempLevel);
        if (tempCost <= availableGp) {
            maxConsLevels++;
            tempLevel++;
        } else {
            break;
        }
    }

    let maxChargeLevels = 0;
    tempCost = 0;
    tempLevel = chargeStart;
    while (tempCost < availableGp) {
        tempCost += getUpgradeCost(CHARGE, tempLevel);
        if (tempCost <= availableGp) {
            maxChargeLevels++;
            tempLevel++;
        } else {
            break;
        }
    }

    let maxPercentDamageLevels = 0;
    tempCost = 0;
    tempLevel = percentDamageStart;
    while (tempCost < availableGp) {
        tempCost += getUpgradeCost(PERCENTDAMAGE, tempLevel);
        if (tempCost <= availableGp) {
            maxPercentDamageLevels++;
            tempLevel++;
        } else {
            break;
        }
    }

    let maxCritLevels = 0;
    tempCost = 0;
    tempLevel = critStart;
    while (tempCost < availableGp) {
        tempCost += getUpgradeCost(CRIT, tempLevel);
        if (tempCost <= availableGp) {
            maxCritLevels++;
            tempLevel++;
        } else {
            break;
        }
    }

    // Limit search space to reasonable values
    // Add 1 to include the current value in our search
    const maxBase = baseStart + maxBaseLevels + 1;
    const maxCons = consStart + maxConsLevels + 1;
    const maxCharge = chargeStart + maxChargeLevels + 1;
    const maxPercentDamage = percentDamageStart + maxPercentDamageLevels + 1;
    const maxCrit = critStart + maxCritLevels + 1;
    console.log(maxBase + " " + maxCons + " " + maxCharge + " " + maxPercentDamage + " " + maxCrit)

    // Brute force through all possible combinations
    for (let b = baseStart; b <= maxBase; b++) {
        for (let c = consStart; c <= maxCons; c++) {
            for (let ch = chargeStart; ch <= maxCharge; ch++) {
                for (let pd = percentDamageStart; pd <= maxPercentDamage; pd++) {
                    for (let cr = critStart; cr <= maxCrit; cr++) {
                        console.log("Trying: " + b + ", " + c + ", " + ch + ", " + pd + ", " + cr);
                        // Skip if no upgrades
                        if (b === baseStart && c === consStart && ch === chargeStart && pd === percentDamageStart && cr === critStart) continue;

                        // Calculate cost for this combination
                        let totalCost = 0;

                        // Base damage cost
                        for (let level = baseStart; level < b; level++) {
                            totalCost += getUpgradeCost(BASEDAMAGE, level);
                        }

                        // Consecutive cost
                        for (let level = consStart; level < c; level++) {
                            totalCost += getUpgradeCost(CONSECUTIVE, level);
                        }

                        // Charge cost
                        for (let level = chargeStart; level < ch; level++) {
                            totalCost += getUpgradeCost(CHARGE, level);
                        }

                        // percent damage cost
                        for (let level = percentDamageStart; level < pd; level++) {
                            totalCost += getUpgradeCost(PERCENTDAMAGE, level);
                        }

                        // crit cost
                        for (let level = critStart; level < cr; level++) {
                            totalCost += getUpgradeCost(CRIT, level);
                        }

                        // Skip if cost exceeds available GP
                        if (totalCost > availableGp) continue;

                        // Calculate damage with this combination
                        const charges = 8 + ch + Math.floor(playerlevel.value / 25);
                        const damage = calcDamage(charges, b, c, pd, cr, critCounter.value, playerlevel.value);

                        // Update best if this is better
                        if (damage > bestDamage) {
                            bestDamage = damage;
                            bestBase = b;
                            bestConsecutive = c;
                            bestCharge = ch;
                            bestPercentDamage = pd;
                            bestCrit = cr;
                        }
                    }
                }
            }
        }
    }
    console.log("done")
    // Update the desired values with the best combination
    desired2Basedamage.value = bestBase;
    desired2Consecutive.value = bestConsecutive;
    desired2Charge.value = bestCharge;
    desired2PercentageDamage.value = bestPercentDamage;
    desired2Crit.value = bestCrit;
}
</script>

<template>
    <div class="container mx-auto px-4 py-8 max-w-full">
        <header class="mb-8">
            <!-- Header content if any -->
        </header>

        <main>
            <!-- Stats Section -->
            <div class="mb-8">
                <h2 class="text-2xl font-bold mb-4 text-gray-800 border-b pb-2">Stats</h2>
                <div class="grid grid-cols-1 md:grid-cols-5 gap-4">
                    <div class="flex items-center">
                        <label class="w-28 font-medium">Bosstier</label>
                        <input type="number" v-model="bosstier" min="1" max="1000" @change="updateEverything"
                               tabindex="1"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                    <div class="flex items-center">
                        <label class="w-28 font-medium">Playerlevel</label>
                        <input type="number" v-model="playerlevel" min="0" max="1000" @change="updateEverything"
                               tabindex="2"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                    <div class="flex items-center">
                        <label class="w-28 font-medium">Current XP</label>
                        <input type="number" v-model="currentXp" min="0" max="100000" @change="updateEverything"
                               tabindex="3"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                    <div class="flex items-center">
                        <label class="w-28 font-medium">Current GP</label>
                        <input type="number" v-model="currentGp" min="0" max="9999999" @change="updateEverything"
                               tabindex="4"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                    <div class="flex items-center">
                        <label class="w-28 font-medium">Critcounter</label>
                        <input type="number" v-model="critCounter" min="0" max="15" @change="updateEverything"
                               tabindex="5"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                </div>
            </div>

            <!-- Buffs Section -->
            <div class="mb-8">
                <h2 class="text-2xl font-bold mb-4 text-gray-800 border-b pb-2">Buffs(%)</h2>
                <div class="grid grid-cols-1 md:grid-cols-5 gap-4">
                    <div class="flex items-center">
                        <label class="w-28 font-medium">Damage</label>
                        <input type="number" v-model="buffDamage" @change="updateEverything" min="0" max="1000"
                               tabindex="6"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                    <div class="flex items-center">
                        <label class="w-28 font-medium">XP</label>
                        <input type="number" v-model="buffXp" @change="updateEverything" min="0" max="1000" tabindex="7"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                    <div class="flex items-center">
                        <label class="w-28 font-medium">GP</label>
                        <input type="number" v-model="buffGp" @change="updateEverything" min="0" max="1000" tabindex="8"
                               class="border rounded px-2 py-1 w-24 focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                    </div>
                </div>
            </div>

            <!-- Upgrades and Planning Section -->
            <div>
                <h2 class="text-2xl font-bold mb-4 text-gray-800 border-b pb-2">Upgrades and Planning</h2>
                <div class="overflow-x-auto">
                    <table class="min-w-full bg-white">
                        <thead>
                        <tr class="bg-gray-100">
                            <th class="px-4 py-2 text-left w-24">Current</th>
                            <th class="px-4 py-2 text-left w-20">Level</th>
                            <th class="px-4 py-2 text-left w-20">Cost</th>
                            <th class="px-4 py-2 text-left w-20">Damage</th>
                            <th class="px-4 py-2 text-left w-20">GP/Dmg</th>
                            <th class="px-4 py-2 text-left w-24 bg-blue-50">Target 1</th>
                            <th class="px-4 py-2 text-left w-20 bg-blue-50 border-r-2 border-blue-200">Cost</th>
                            <th class="px-4 py-2 text-left w-24 bg-green-50">Target 2</th>
                            <th class="px-4 py-2 text-left w-20 bg-green-50">Cost</th>
                        </tr>
                        </thead>
                        <tbody>
                        <!-- Base damage row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">Base damage</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="basedamage" @change="updateEverything" min="0" max="9999"
                                       tabindex="9"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500">
                            </td>
                            <td class="px-4 py-2">{{ basedamageCost }}</td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': damageAfterBaseUpgrade === highestDamage }">
                                {{ damageAfterBaseUpgrade }}
                            </td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': basedamageEfficiency === bestEfficiency }">
                                {{ formattedBasedamageEfficiency }}
                            </td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredBasedamage" @change="updateEverything" min="0"
                                       max="9999" tabindex="16"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500">
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(BASEDAMAGE, basedamage, desiredBasedamage) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2Basedamage" @change="updateEverything" min="0"
                                       max="9999" tabindex="23"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500">
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(BASEDAMAGE, basedamage, desired2Basedamage) }}
                            </td>
                        </tr>

                        <!-- Percentage damage row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">Percent damage</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="percentdamage" @change="updateEverything" min="0" max="9999"
                                       tabindex="10"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2">{{ percentdamageCost }}</td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': damageAfterPercentdamageUpgrade === highestDamage }">
                                {{ damageAfterPercentdamageUpgrade }}
                            </td>
                            <td class="px-4 py-2" :class="{ 'text-green-600': percentdamageEfficiency === bestEfficiency }">
                                {{ formattedPercentageDamageEfficiency }}
                            </td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredPercentageDamage" @change="updateEverything" min="0"
                                       max="9999" tabindex="17"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(PERCENTDAMAGE, percentdamage, desiredPercentageDamage) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2PercentageDamage" @change="updateEverything" min="0"
                                       max="9999" tabindex="24"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(PERCENTDAMAGE, percentdamage, desired2PercentageDamage) }}
                            </td>
                        </tr>

                        <!-- Crit row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">Crit</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="crit" @change="updateEverything" min="0" max="14"
                                       tabindex="11"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2">{{ critCost }}</td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': damageAfterCritUpgrade === highestDamage }">
                                {{ damageAfterCritUpgrade }}
                            </td>
                            <td class="px-4 py-2" :class="{ 'text-green-600': critEfficiency === bestEfficiency }">
                                {{ critEfficiency }}
                            </td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredCrit" @change="updateEverything" min="0"
                                       max="9999" tabindex="18"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(CRIT, crit, desiredCrit) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2Crit" @change="updateEverything" min="0"
                                       max="9999" tabindex="25"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(CRIT, crit, desired2Crit) }}
                            </td>
                        </tr>

                        <!-- Consecutive row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">Consecutive</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="consecutive" @change="updateEverything" min="0" max="9999"
                                       tabindex="12"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2">{{ consecutiveCost }}</td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': damageAfterConsecutiveUpgrade === highestDamage }">
                                {{ damageAfterConsecutiveUpgrade }}
                            </td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': consecutiveEfficiency === bestEfficiency }">
                                {{ formattedConsecutiveEfficiency }}
                            </td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredConsecutive" @change="updateEverything" min="0"
                                       max="9999" tabindex="19"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(CONSECUTIVE, consecutive, desiredConsecutive) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2Consecutive" @change="updateEverything" min="0"
                                       max="9999" tabindex="26"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(CONSECUTIVE, consecutive, desired2Consecutive) }}
                            </td>
                        </tr>

                        <!-- GP row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">GP</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="gp" @change="updateEverything" min="0" max="9999"
                                       tabindex="13"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2">{{ gpCost }}</td>
                            <td class="px-4 py-2">-</td>
                            <td class="px-4 py-2">-</td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredGp" @change="updateEverything" min="0" max="9999"
                                       tabindex="20"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(GP, gp, desiredGp) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2Gp" @change="updateEverything" min="0" max="9999"
                                       tabindex="27"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(GP, gp, desired2Gp) }}
                            </td>
                        </tr>

                        <!-- XP row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">XP</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="xp" @change="updateEverything" min="0" max="9999"
                                       tabindex="14"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2">{{ xpCost }}</td>
                            <td class="px-4 py-2">-</td>
                            <td class="px-4 py-2">-</td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredXp" @change="updateEverything" min="0" max="9999"
                                       tabindex="21"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(XP, xp, desiredXp) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2Xp" @change="updateEverything" min="0" max="9999"
                                       tabindex="28"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(XP, xp, desired2Xp) }}
                            </td>
                        </tr>

                        <!-- Charge row -->
                        <tr class="border-b">
                            <td class="px-4 py-2 font-medium">Charge</td>
                            <td class="px-4 py-2">
                                <input type="number" v-model="charge" @change="updateEverything" min="0" max="9999"
                                       tabindex="15"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2">{{ chargeCost }}</td>
                            <td class="px-4 py-2"
                                :class="{ 'text-green-600': damageAfterChargeUpgrade === highestDamage }">
                                {{ damageAfterChargeUpgrade }}
                            </td>
                            <td class="px-4 py-2" :class="{ 'text-green-600': chargeEfficiency === bestEfficiency }">
                                {{ formattedChargeEfficiency }}
                            </td>
                            <td class="px-4 py-2 bg-blue-50">
                                <input type="number" v-model="desiredCharge" @change="updateEverything" min="0"
                                       max="9999" tabindex="22"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ calculateUpgradeCost(CHARGE, charge, desiredCharge) }}
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <input type="number" v-model="desired2Charge" @change="updateEverything" min="0"
                                       max="9999" tabindex="29"
                                       class="border rounded px-2 py-1 w-full focus:outline-none focus:ring-2 focus:ring-blue-500"/>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ calculateUpgradeCost(CHARGE, charge, desired2Charge) }}
                            </td>
                        </tr>




                        <!-- Total Cost row -->
                        <tr class="border-b bg-gray-50">
                            <td colspan="5" class="px-4 py-2"></td>
                            <td class="px-4 py-2 font-bold bg-blue-50">Total Cost</td>
                            <td class="px-4 py-2 font-bold bg-blue-50 border-r-2 border-blue-200"
                                :class="{ 'text-green-600': totalUpgradeCost <= currentGp, 'text-red-500': totalUpgradeCost > currentGp }">
                                {{ totalUpgradeCost }}
                            </td>
                            <td class="px-4 py-2 font-bold bg-green-50">Total Cost</td>
                            <td class="px-4 py-2 font-bold bg-green-50"
                                :class="{ 'text-green-600': totalUpgradeCost2 <= currentGp, 'text-red-500': totalUpgradeCost2 > currentGp }">
                                {{ totalUpgradeCost2 }}
                            </td>
                        </tr>

                        <!-- Total Damage row -->
                        <tr class="border-b">
                            <td colspan="4" class="px-4 py-2 font-bold">Total Damage</td>
                            <td class="px-4 py-2">{{ totalDamage }}</td>
                            <td class="px-4 py-2 bg-blue-50"></td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ totalDesiredDamage }}
                                <span class="ml-2">(+{{
                                        totalDesiredDamage - totalDamage
                                    }} | +{{ damageIncreasePercentage }}%)</span>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                <button @click="optimizeUpgrades" tabindex="30"
                                        class="bg-green-500 hover:bg-green-600 text-white px-3 py-1 rounded focus:outline-none focus:ring-2 focus:ring-green-500">
                                    Optimize
                                </button>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ totalDesiredDamage2 }}
                                <span class="ml-2 ">(+{{
                                        totalDesiredDamage2 - totalDamage
                                    }} | +{{ damageIncreasePercentage2 }}%)</span>
                            </td>
                        </tr>

                        <!-- Gained XP row -->
                        <tr class="border-b">
                            <td colspan="4" class="px-4 py-2 font-bold">Gained XP</td>
                            <td class="px-4 py-2">{{ rewardXp }}</td>
                            <td class="px-4 py-2 bg-blue-50"></td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ desiredRewardXp }}
                                <span class="ml-2">(+{{ desiredRewardXp - rewardXp }} | +{{
                                        xpIncreasePercentage
                                    }}%)</span>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                Rank <select v-model="rank" @change="updateEverything">
                                <option value="0">2500+</option>
                                <option value="2">1001+</option>
                                <option value="3">251+</option>
                                <option value="4">51+</option>
                                <option value="5">11+</option>
                                <option value="6">4+</option>
                                <option value="8">2+</option>
                                <option value="10">1</option>
                            </select>
                            </td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ desired2RewardXp }}
                                <span class="ml-2">(+{{ desired2RewardXp - rewardXp }} | +{{
                                        xpIncreasePercentage2
                                    }}%)</span>
                            </td>
                        </tr>

                        <!-- Gained GP row -->
                        <tr class="border-b">
                            <td colspan="4" class="px-4 py-2 font-bold">Gained GP</td>
                            <td class="px-4 py-2">{{ rewardGp }}</td>
                            <td class="px-4 py-2 bg-blue-50"></td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">
                                {{ desiredRewardGp }}
                                <span class="ml-2">(+{{ desiredRewardGp - rewardGp }} | +{{
                                        gpIncreasePercentage
                                    }}%)</span>
                            </td>
                            <td class="px-4 py-2 bg-green-50"></td>
                            <td class="px-4 py-2 bg-green-50">
                                {{ desired2RewardGp }}
                                <span class="ml-2">(+{{ desired2RewardGp - rewardGp }} | +{{
                                        gpIncreasePercentage2
                                    }}%)</span>
                            </td>
                        </tr>
                        <tr class="border-b">
                            <td colspan="4" class="px-4 py-2 font-bold">Level after Boss</td>
                            <td class="px-4 py-2">{{ levelAfterBoss }}</td>
                            <td class="px-4 py-2 bg-blue-50"></td>
                            <td class="px-4 py-2 bg-blue-50 border-r-2 border-blue-200">{{ levelAfterBossDesired }}</td>
                            <td class="px-4 py-2 bg-green-50"></td>
                            <td class="px-4 py-2 bg-green-50">{{ levelAfterBossDesired2 }}</td>
                        </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </main>
    </div>
</template>