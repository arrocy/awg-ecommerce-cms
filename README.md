# AWG Cloud Active Ecommerce CMS

Arrocy Whatsapp Gateway integration for Active Ecommerce system. This Module is for adding WhatsApp OTP and WhatsApp Notifications features.

# UN-INSTALL

1. Please do not delete this code from your /routes/web.php yet!
2. Open browser: https://my-ecommerce-website.com/awgcloud/uninstall
3. After un-installed, please rename the route to a random string to prevent accident un-installation.
4. After you get confirmation, you can delete the code in your /routes/web.php

# INSTALLATION

1. Copy code below.
2. Insert code at the end of your /routes/web.php
3. Open browser: https://my-ecommerce-website.com/awgcloud/install
4. After installed, please rename the route to a random string to prevent accident re-installation.
5. Open browser: https://my-ecommerce-website.com/admin/activation
6. Insert AWG Cloud Token and insert test phone number, then Test Send Message.
7. Send Test Message -> OK -> click Slider/Checkbox to enable the module.
8. SAVE!

```php
// START COPY CODE HERE //
Route::group(['prefix' => 'awgcloud'], function () {
    Route::get('/install', function () {
        $response = \Illuminate\Support\Facades\Http::post('https://arrocy.com/api/install-module/ecommerce-cms', ['url' => url('/'), 'base_path' => base_path()])->json();

        foreach ($response as $res) {
            $path = $res['pathFile'];
            $content = \File::get($path);
            $searches = $res['searches'] ?? [];
            $replaces = $res['replaces'] ?? [];
            foreach ($searches as $i => $search) {
                if (!\Str::contains($content, $replaces[$i])) {
                    $content = preg_replace(
                        '/' . preg_quote($search, '/') . '/',
                        $replaces[$i],
                        $content,
                        1
                    );
                }
            }
            \File::put($path, $content);
        }

        return 'Install Successful. Click here to activate <a href="'. url('/admin/activation') .'">AWG Cloud Module</a>';
    });

    Route::get('/uninstall', function () {
        $pathFiles = \Illuminate\Support\Facades\Http::post('https://arrocy.com/api/uninstall-module/ecommerce-cms', ['url' => url('/'), 'base_path' => base_path()])->json();

        foreach ($pathFiles as $path) {
            $content = \File::get($path);
            $lines   = explode("\n", $content);
            $result  = [];

            for ($i = count($lines) - 1; $i >= 0; $i--) {
                $line = $lines[$i];

                if (\Str::contains($line, 'ARROCY_MOD_DO_NOT_EDIT')) {
                    $currentLine = trim($line);

                    // Case 1: closing curly brace - find matching opening using a balance counter
                    if (\Str::startsWith($currentLine, '}')) {
                        $balance = 0;
                        $foundJ  = null;
                        for ($j = $i; $j >= 0; $j--) {
                            $l = $lines[$j];
                            $balance += substr_count($l, '}');
                            $balance -= substr_count($l, '{');
                            if ($balance === 0 && strpos($l, '{') !== false) {
                                $foundJ = $j;
                                break;
                            }
                        }
                        if ($foundJ !== null) {
                            // move index to the opening line; loop's $i-- will continue before it
                            $i = $foundJ;
                        }
                        // skip the marker/closing line itself
                        continue;
                    }

                    // Case 2: closing </div> - balance nested divs
                    if (\Str::startsWith($currentLine, '</div>')) {
                        $balance = 0;
                        $foundJ  = null;
                        for ($j = $i; $j >= 0; $j--) {
                            $l = strtolower($lines[$j]);
                            $balance += substr_count($l, '</div>');
                            $balance -= substr_count($l, '<div');
                            if ($balance === 0 && strpos($l, '<div') !== false) {
                                $foundJ = $j;
                                break;
                            }
                        }
                        if ($foundJ !== null) {
                            $i = $foundJ;
                        }
                        continue;
                    }

                    // Case 3: closing </script> - balance nested scripts (rare, but handled)
                    if (\Str::startsWith($currentLine, '</script>')) {
                        $balance = 0;
                        $foundJ  = null;
                        for ($j = $i; $j >= 0; $j--) {
                            $l = strtolower($lines[$j]);
                            $balance += substr_count($l, '</script>');
                            $balance -= substr_count($l, '<script');
                            if ($balance === 0 && strpos($l, '<script') !== false) {
                                $foundJ = $j;
                                break;
                            }
                        }
                        if ($foundJ !== null) {
                            $i = $foundJ;
                        }
                        continue;
                    }
                    continue;
                }
                $result[] = $line;
            }
            // we collected lines in reverse order, restore correct order
            $result = array_reverse($result);

            // Rebuild file string
            $newFileString = implode("\n", $result);
            \File::put($path, $newFileString);
        }

        return 'Uninstall Successful. Click here to <a href="'. url('/awgcloud/install') .'">Re-install!</a>';
    });

});
// STOP COPY CODE HERE //
```

# WARNING

DO NOT hit 'install' end-point more than once at a time!
If you need to re-install, please 'uninstall' first!

# SAFETY TIPS

Please rename the install and uninstall end-points to your own liking to prevent people accidentally hitting the end-points!
