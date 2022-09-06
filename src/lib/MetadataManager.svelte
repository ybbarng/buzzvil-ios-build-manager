<script>
  const input_hint = `[
  {
    "module": "BuzzResource",
    "version": "0.13.0",
    "build_number": "1300"
  },
  {
    "module": "BuzzAdBenefitBase",
    "version": "3.9.0",
    "build_number": "40900"
  },
  {
    "module": "BuzzAdBenefitNative",
    "version": "3.9.0",
    "build_number": "40900"
  },
  {
    "module": "BuzzAdBenefitInterstitial",
    "version": "3.9.0",
    "build_number": "40900"
  },
  {
    "module": "BuzzAdBenefitFeed",
    "version": "3.9.0",
    "build_number": "40900"
  },
  {
    "module": "BuzzAdBenefit",
    "version": "3.9.0",
    "build_number": "40900"
  },
  {
    "module": "BABSample",
    "version": "3.7.1",
    "build_number": "40701"
  }
]`;
  var modules = [];
  var error_message = null;
  var input = "";
  var output = "";
  const updateModules = (new_modules) => {
    new_modules.forEach((module_version) => {
      modules.push({
        version: module_version,
        update: "exclude",
      });
    });
  }
  const onInputChanged = () => {
    modules = [];
    error_message = null;
    if (input.length === 0) {
      return;
    }
    try {
      updateModules(JSON.parse(input));
    } catch(e) {
      if (e instanceof SyntaxError) {
        error_message = "올바른 JSON 객체를 입력하세요.";
      } else {
        console.error(e);
        error_message = "알 수 없는 에러가 발생했습니다.";
      }
    }
  }
  const onUpdateClicked = (i, updateType) => {
    const module = modules[i];
    if (module === undefined) {
      return;
    }
    if (module.update === updateType) {
      module.update = "exclude";
    } else {
      module.update = updateType;
    }
  }
  const update = (version, update) => {
    const newVersion = {
      ...version
    };
    const versions = newVersion.version.split('.').map(Number);
    var build_number = parseInt(newVersion.build_number);
    switch (update) {
      case "major":
        versions[0] += 1;
        build_number += 10000;
        break;
      case "minor":
        versions[1] += 1;
        build_number += 100;
        break;
      case "patch":
        versions[2] += 1;
        build_number += 1;
        break;
    }
    newVersion.version = versions.join('.');
    newVersion.build_number = build_number.toString();
    return newVersion;
  }
  const buildOutput = () => {
    const output = [];
    modules.forEach((module) => {
        if (module.update === "exclude") {
          return;
        }
        output.push(update(module.version, module.update));
    });
    return output;
  }
  const onUpdateChanged = () => {
    output = JSON.stringify(buildOutput(), null, 2);
  }
</script>

<div class="metadata-manager">
  <div class="input">
    <div class="title">
      <h2>1. 현재 모듈의 버전 정보를 입력하세요</h2>
    </div>
    <textarea
      name="input" cols="40" rows="40"
      placeholder={input_hint}
      bind:value={input}
      on:input={onInputChanged}
      />
  </div>
  {#if error_message === null}
    <div class="manage">
      <div class="title">
        <h2>2. 각 모듈의 업데이트 정보를 설정하세요.</h2>
      </div>
      <div class="modules">
        {#each modules as module, i}
          <div class="module">
            <div class="version">
              <span class="name" title="module">
                {module.version.module}
              </span>
              <span class="tag semver" title="version">
                {module.version.version}
              </span>
              <span class="tag build-number" title="build_number">
                {module.version.build_number}
              </span>
            </div>
            <div class="update">
              <div>
                <input
                  id={"exclude" + i}
                  type="radio"
                  bind:group={module.update}
                  value="exclude"
                  on:change={onUpdateChanged}
                  >
                <label class="button" for={"exclude" + i}>제외</label>
              </div>
              <div>
                <input
                  id={"major" + i}
                  type="radio"
                  bind:group={module.update}
                  value="major"
                  on:change={onUpdateChanged}
                  >
                <label class="button" for={"major" + i}>Major</label>
              </div>
              <div>
                <input
                  id={"minor" + i}
                  type="radio"
                  bind:group={module.update}
                  value="minor"
                  on:change={onUpdateChanged}
                  >
                <label class="button" for={"minor" + i}>Minor</label>
              </div>
              <div>
                <input
                  id={"patch" + i}
                  type="radio"
                  bind:group={module.update}
                  value="patch"
                  on:change={onUpdateChanged}
                  >
                <label class="button" for={"patch" + i}><span>Patch</label>
              </div>
            </div>
          </div>
        {/each}
      </div>
    </div>
  {:else}
    <div class="error">
      <div class="title">
        <h2>2. 에러가 발생했습니다.</h2>
      </div>
      <div>
      <p>{error_message}</p>
      </div>
    </div>
  {/if}
  <div class="output">
    <div class="title">
      <h2>3. 결과를 확인하고 복사하여 사용하세요.</h2>
    </div>
    <textarea name="output" cols="40" rows="40"
      bind:value={output} />
  </div>
</div>

<style>
.metadata-manager {
  display: flex;
  flex-direction: row;
  gap: 3em;
}
.modules {
  display: flex;
  flex-direction: column;
  gap: 1em;

}
.module {
  border: 1px solid #eee;
  border-radius: 10px;
  padding: 1em;
}
.module .version {
  display: flex;
  flex-direction: row;
  margin-bottom: 0.5em;
}
.module .name {
  font-weight: bold;
  margin-right: auto;
}
.module .tag {
  border: 1px solid black;
  border-radius: 5px;
  padding: 0.1em 0.8em;
  font-size: 10px;
  margin-left: 0.5em;
}
.module .semver {
  border-color: #142024;
  background: #29424a;
}

.module .build-number {
  border-color: #111f18;
  background: #274738;
}
.module input {
  opacity: 0;
  width: 0;
  height: 0;
  display: none;
}
.update {
  display: flex;
  flex-direction: row;
}
.button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  background-color: #1a1a1a;
  cursor: pointer;
  transition: border-color 0.25s;
  white-space:nowrap;
  display:inline-block;
}
.button:hover {
  border-color: #646cff;
}
input:checked + .button {
  outline: 4px auto #99c8ff;
  background: #3e5269;

}
</style>
