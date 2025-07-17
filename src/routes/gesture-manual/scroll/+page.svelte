<script>
    import GestureUI from "../GestureUI.svelte";
    import ScrollCode from "./ScrollCode.svelte";
</script>


<GestureUI enableDrag={false} enableClick={true} enableScroll={true}  enableCursor={true}/>

<main class="flex-1 max-w-6xl mx-auto px-4 justify-center">
    <h2 class="py-2 text-6xl text-center">Scrolling Gesture</h2>
    <p class="text-lg mt-10">
    This demo lets you explore information about fighter aircraft using only hand gesture.
    </p>
    <ul class="list-disc px-10 text-lg">
    <li>Your <strong>right hand</strong> acts as a virtual mouse pointer using real-time hand tracking.</li>
    <li>Flick your <strong>Index finger</strong> down to scroll the page downwards.</li>
    <li>Flick your  <strong>Index finger</strong> and <strong> Middle finder </strong> up scroll the page upwards.</li>
    </ul>

    <section class="text-center mt-10 flex flex-rows-2">
    <img src="/video-demos/Scroll-down.gif" alt="Scroll Down Gesture"
    class="rounded-xl shadow-lg w-[400px] mx-auto" />

    <img src="/video-demos/Scroll-up.gif" alt="Scroll Down Gesture"
    class="rounded-xl shadow-lg w-[400px] mx-auto" />
    </section>

    <ScrollCode/>

    <h1 class="text-5xl py-5 text-center font-semibold ">Scroll Demo: Fighter Aircraft: Masters of the Sky</h1>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">The Mission of a Fighter</h2>
    <p class="text-lg px-4">
        Fighter aircraft are military planes built to win control of the air. Their job is simple in theory — engage enemy aircraft and secure air superiority — but achieving this has pushed the edge of engineering and human capability for over a century. From early canvas-and-wood biplanes to radar-evading stealth jets, fighter design has always raced ahead of the curve.
    </p>
    <p class="text-lg px-4 mt-2">
        To dominate aerial combat, a fighter must be fast, nimble, and lethal. But it's not just about speed or firepower. Dogfighting, radar detection, fuel efficiency, weapon integration, and pilot survivability all converge into one of the most demanding design challenges in defense technology.
    </p>
    </section>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">Birth of Aerial Combat</h2>
    <p class="text-lg px-4">
        When World War I broke out in 1914, aircraft were mostly scouts — unarmed, fragile observers flying above the trenches. But within weeks, pilots began arming themselves with pistols and rifles to fire at enemy planes. Quickly, engineers recognized the need to create dedicated fighters. Designs like the French Voisin and the British Sopwith Camel ushered in a new kind of warfare: dogfighting in the sky. By mid-war, aircraft were equipped with synchronized machine guns firing through the propeller arc — a leap in lethality that defined early air combat.
    </p>
    <p class="text-lg px-4 mt-2">
        The Germans’ Fokker Eindecker gave them temporary aerial dominance during the so-called "Fokker Scourge," but the Allies soon responded with better training, tactics, and machines. By 1918, most major powers had specialized single-seat fighters, capable of 100+ mph, fighting vicious battles for control of the airspace. The era's biplanes were built from wood and fabric, often fragile, and lacked safety systems — no parachutes, no ejection seats. A single hit to the fuel line or structure could mean certain death.
    </p>
    </section>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">Between Wars: Design Innovation and Stagnation</h2>
    <p class="text-lg px-4">
        The interwar years saw aviation technologies split across continents. While France and the Soviet Union invested heavily in metal monoplane designs, Britain and the U.S. held onto biplanes for longer. Many planes were still fabric-covered, and designs prioritized agility over speed or firepower. However, important innovations were brewing. Aircraft racing competitions like the Schneider Trophy pushed nations to experiment with streamlining, retractable landing gear, and higher-powered engines. These technologies would become standard in fighters by World War II.
    </p>
    <p class="text-lg px-4 mt-2">
        Heavy fighters also gained attention. These were twin-engine aircraft, often adapted from bombers, designed for long-range patrols or bomber escort. But in combat, they struggled against nimble single-engine adversaries and were frequently reassigned to night fighting, reconnaissance, or ground attack. Still, they helped pave the way for multirole fighter concepts that would dominate later decades.
    </p>
    </section>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">World War II: Fighter Aircraft Come of Age</h2>
    <p class="text-lg px-4">
        By the time WWII began, fighter aircraft had become all-metal monoplanes armed with multiple machine guns or cannons. Engines were more powerful, often reaching over 1,000 horsepower, and aircraft were capable of climbing, diving, and turning at incredible rates. The Battle of Britain demonstrated the strategic power of fighters — Spitfires and Hurricanes defending the UK from German bombers changed the war’s course. On the Eastern Front, Soviet fighters initially lagged, but newer designs like the Yak-9 and La-5 eventually proved formidable.
    </p>
    <p class="text-lg px-4 mt-2">
        Meanwhile, the Pacific saw Japanese fighters like the A6M Zero dominate early with superior maneuverability. But the Allies learned to use tactics and heavier firepower to counter them. The introduction of the P-38 Lightning, F4U Corsair, and P-51 Mustang marked a turning point, giving Allied pilots the tools to regain control. The Mustang, in particular, could escort bombers deep into enemy territory, helping to dismantle the Luftwaffe’s defense.
    </p>
    <p class="text-lg px-4 mt-2">
        WWII also saw the birth of the jet fighter. Germany’s Me 262 was the first operational jet-powered aircraft, capable of speeds that outclassed every Allied plane. However, production delays and fuel shortages limited its impact. The war ended with jet engines poised to transform aviation.
    </p>
    </section>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">Jet Age and the Cold War</h2>
    <p class="text-lg px-4">
        After WWII, piston engines quickly gave way to jets. The Korean War marked the first jet-versus-jet battles, with the MiG-15 and F-86 Sabre squaring off in high-speed duels. These aircraft introduced swept wings, radar systems, and infrared missiles. The Cold War then triggered rapid advancements in speed, altitude, and armament, with interceptors like the MiG-21 and F-104 Starfighter designed to shoot down nuclear bombers at extreme altitudes.
    </p>
    <p class="text-lg px-4 mt-2">
        Meanwhile, multirole fighters emerged to handle both air combat and ground strike missions. The F-4 Phantom II exemplified this shift, with twin engines, long-range radar, and the ability to carry missiles, bombs, and cannons. Electronic warfare, data links, and guided munitions became essential, and fighter designs evolved from dogfighters into highly adaptable platforms for a variety of battlefield roles.
    </p>
    </section>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">Technology and Tactics Transform</h2>
    <p class="text-lg px-4">
        As radar and missile technology improved, beyond-visual-range (BVR) combat became a dominant strategy. Early assumptions that guns were obsolete proved false during Vietnam, where close-range fights still occurred. The U.S. Navy created the famous TOPGUN program to teach advanced maneuvering and missile usage. Over time, fighters began to blend BVR capability with close-in agility.
    </p>
    <p class="text-lg px-4 mt-2">
        Stealth became the next leap forward. Reducing radar cross-section meant aircraft could strike undetected. The F-117 Nighthawk began the trend, but the F-22 Raptor became the first true stealth air superiority fighter, capable of defeating enemies before being seen. Composite materials, internal weapon bays, and heat signature control all became vital design elements.
    </p>
    </section>

    <section class="mt-6">
    <h2 class="text-3xl font-semibold py-2">Fifth Generation and Beyond</h2>
    <p class="text-lg px-4">
        The current era is dominated by fifth-generation fighters like the F-22 and F-35, the Chinese J-20, and Russia’s Su-57. These jets combine stealth, sensor fusion, supercruise (supersonic speed without afterburners), and electronic warfare. They’re less about raw speed and more about dominating information, coordinating with drones, satellites, and surface assets to out-think and out-shoot the enemy.
    </p>
    <p class="text-lg px-4 mt-2">
        The F-35, in particular, is redefining multirole capability. With over 3,000 planned deliveries, it is set to become the most widely used fighter of its era. It can switch between air-to-air, air-to-ground, and intelligence roles with ease, using its sensors and stealth to support allied forces without ever firing a shot.
    </p>
    </section>

    <section class="mt-6 mb-10">
    <h2 class="text-3xl font-semibold py-2">Why Fighter Aircraft Still Matter</h2>
    <p class="text-lg px-4">
        Despite drones, missiles, and cyberwarfare, fighter jets remain essential. They offer flexible response, deep strike ability, and a visible show of force. In any major conflict, the first battle is often for the skies — and those who control the air can shape the battlefield below. As technology evolves, fighter aircraft continue to lead that charge.
    </p>
    </section>

    <footer class="mt-10 text-center text-sm text-gray-500">
    &copy; 2025 Fighter Aircraft Project — Educational Use Only.
    </footer>
</main>

