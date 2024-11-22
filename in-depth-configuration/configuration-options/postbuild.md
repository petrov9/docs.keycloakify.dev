# postBuild

{% hint style="info" %}
Only available in Vite projects, not in Webpack
{% endhint %}

The postBuild hook is called just before Keycloakify bundles the themes resources into the jar.

This gives you the ability to implement some custom transformation.

Let's say, for example, we have a big `material-icons` in our `public` directory and those icons are used in the main app but not in the Keycloak theme. We can use the postBuild hook to make sure that those icons are not bundled in the generated jar files.

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import * as fs from "fs/promises";
import * as path from "path";

export default defineConfig({
    plugins: [
        react(),
        keycloakify({
<strong>            postBuild: async (buildContext) => {
</strong><strong>                await fs.rm(
</strong><strong>                    path.join(
</strong><strong>                        "theme",
</strong><strong>                        buildContext.themeNames[0], // keycloakify-starter
</strong><strong>                        "login", // Note: We assume we only have an login theme, if we had an account theme we would have to remove it there as well.
</strong><strong>                        "resources",
</strong><strong>                        "dist", // Your Vite dist/ or Webpack build/ is here.
</strong><strong>                        "material-icons"
</strong><strong>                    ),
</strong><strong>                    { recursive: true }
</strong><strong>                );
</strong><strong>            }
</strong>        })
    ]
});
</code></pre>

When this function is invoked the current working directory (process.cwd()) is the root of the directory of the files about to be archived.

You can get an idea of how is structured the files inside the jar by extracting it manually:

```bash
mkdir dist_keycloak/extracted
cd dist_keycloak/extracted
jar -xf ../keycloak-theme-for-kc-25-and-above.jar
```

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption><p>Overview of the content of the jar extracted</p></figcaption></figure>

### Debugging

Note that the script is executed in a different thread. console.log() won't work.  \
If you want to debug you can write your logs into a file. Example:  \


{% code title="vite.config.ts" %}
```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { keycloakify } from "keycloakify/vite-plugin";

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [
        react(),
        keycloakify({
            accountThemeImplementation: "Single-Page",
            postBuild: async (buildContext) => {

                const fs = await import("fs");
                const path = await import("path");

                const logFilePath = path.join(buildContext.projectDirPath, "postBuildLog.txt");

                fs.rmSync(logFilePath, { force: true });

                const log= (msg: string) => {
                    fs.appendFileSync(
                        logFilePath,
                        Buffer.from(msg + "\n", "utf8")
                    );
                };

                // This will be logged into postBuildLog.txt at the root of your project.
                log("Hello World");

            }
        })
    ]
});

```
{% endcode %}