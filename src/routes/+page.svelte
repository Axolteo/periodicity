<script lang="ts">
  import { periodicTable } from "$lib/periodic";
  let phrase: string = "";
  let periodic: string = "";
  let elmName: string = "";
  let elmSymbol: string = "";

  interface searchResult {
    depth: number;
    value: string[];
  }
  function phraseToPeriodic(phrase: string) {
    if (phrase == "") {
      return "No phrase entered";
    }
    const cleaned = phrase.toLowerCase();
    console.log(cleaned);

    let symbols = [];
    for (let i = 0; i < cleaned.length; i++) {
      let gramOne = cleaned.substring(i, i + 1);
      let gramTwo = cleaned.substring(i, i + 2);

      if (symbolToName(gramTwo) != null) {
        symbols.push(symbolToName(gramTwo));
        i++;
      } else if (symbolToName(gramOne) != null) {
        symbols.push(symbolToName(gramOne));
      } else {
        symbols.push(recursePeriodic(gramOne, 0, []).value);
      }
    }

    console.log(symbols);

    let result = "";
    for (let i = 0; i < symbols.length; i++) {
      if (typeof symbols[i] === "string") {
        result += `${symbols[i]} `;
      } else {
        result += `${nestedFormat(symbols[i] as string[])} `;
      }
    }
    console.log(result);
    return result;
  }

  function periodicToPhrase(periodic: string) {
    if (periodic.trim() === "") {
      return "No elements entered";
    }

    let cleaned = periodic.replace(/\b[a-z]+\b/gi, (match) => {
      const elm = $periodicTable.find(
        (elm) => elm.element.toLowerCase() === match.toLowerCase()
      );
      return elm ? elm.symbol : match;
    });

    cleaned = cleaned
      .replace(/ {3,}/g, "<<<SPACE>>>")
      .replace(/ /g, "")
      .replace(/<<<SPACE>>>/g, " ");

    while (cleaned.includes("(")) {
      let openIndex = -1;
      for (let i = 0; i < cleaned.length; i++) {
        if (cleaned[i] === "(") {
          openIndex = i;
        } else if (cleaned[i] === ")") {
          const closeIndex = i;
          const group = cleaned.substring(openIndex + 1, closeIndex);

          const stack = group.split("-").map((s) => s.trim());
          const result = subtract(stack);

          cleaned =
            cleaned.substring(0, openIndex) +
            result +
            cleaned.substring(closeIndex + 1);
          break;
        }
      }
    }

    console.log(cleaned);
    return cleaned;
  }

  function nestedFormat(elements: string[]): string {
    if (elements.length === 1) return elements[0];

    let result = elements[elements.length - 1];

    result = `${elements[elements.length - 2]} - ${result}`;

    for (let i = elements.length - 3; i >= 0; i--) {
      result = `${elements[i]} - (${result})`;
    }

    return `(${result})`;
  }

  function recursePeriodic(
    char: string,
    depth: number,
    value: string[]
  ): searchResult {
    let candidates = $periodicTable.filter(
      (elm) =>
        elm.symbol.startsWith(char.toLowerCase()) && elm.symbol.length === 2
    );

    if (candidates.length === 0) {
      return { value: [...value, char], depth: Infinity };
    }

    let minRes: searchResult = { value: ["not found"], depth: Infinity };

    for (let i = 0; i < candidates.length; i++) {
      let gramOne = candidates[i].symbol.substring(1);
      let name = symbolToName(gramOne);
      let currentValue = [...value, candidates[i].element];

      if (name != null) {
        return { value: [...currentValue, name], depth: depth };
      } else {
        let res = recursePeriodic(gramOne, depth + 1, currentValue);
        if (res.depth < minRes.depth) {
          minRes = res;
        }
      }
    }

    return {
      value: minRes.value,
      depth: minRes.depth,
    };
  }

  function subtract(stack: string[]): string {
    if (stack.length > 1) {
      let next = stack.slice(1);

      return stack[0].replace(subtract(next), "");
    } else {
      return stack[0];
    }
  }

  function nameToSymbol(name: string) {
    return $periodicTable.find(
      (element) => element.element == name.toLowerCase()
    )?.symbol;
  }

  function symbolToName(symbol: string) {
    return $periodicTable.find(
      (element) => element.symbol == symbol.toLowerCase()
    )?.element;
  }
</script>

<div
  class="w-screen top-0 left-0 h-screen lg:w-[50vw] p-8 flex flex-col justify-start items-stretch gap-12"
>
  <h1>Periodicity</h1>
  <div id="github">
    <!-- <a
      href="https://github.com/Axolteo/axolteo.github.io/tree/main/EquGen"
      target="_blank"
      rel="noopener noreferrer"
    >
      <img src="assets/github logo.svg" />
    </a>-->
  </div>
  <div class="flex flex-col gap-8 overflow-y-scroll">
    <div class="w-full flex flex-col justify-start gap-3">
      <p class="font-medium">Convert a phrase to elements</p>
      <input type="text" bind:value={phrase} placeholder="The phrase to encode" />
      <textarea readonly>{phraseToPeriodic(phrase)}</textarea>
    </div>
    <div class="w-full flex flex-col justify-start gap-3">
      <p class="font-medium">Convert elements to a phrase</p>
      <input type="text" bind:value={periodic} placeholder="The elements to decode" />
      <textarea readonly>{periodicToPhrase(periodic)}</textarea>
    </div>
    <!--<div class="w-full flex flex-col justify-start gap-3">
      <p class="font-medium">Get element name from symbol</p>
      <input type="text" bind:value={elmSymbol} placeholder="Get elm name" />
      <textarea readonly>{symbolToName(elmSymbol)}</textarea>
    </div>
    <div class="w-full flex flex-col justify-start gap-3">
      <p class="font-medium">Get element symbol from name</p>
      <input type="text" bind:value={elmName} placeholder="Get elm symbol" />
      <textarea readonly>{nameToSymbol(elmName)}</textarea>
    </div>-->
  </div>
</div>
<div
  class="lg:visible invisible w-[50vw] h-screen absolute top-0 right-0 flex justify-center items-center
"
>
  <img src="logo.svg" width="480px" height="480px" alt="logo" />
</div>
