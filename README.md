# GDExtension to Module

Port your GDExtension addon to modules with minimal changes. The idea is that you will create a wrapper module that statically links with your addon, and for this you also need to build your addon statically. Thats it.

# Template of Wrapper Module

This repo contains a sample wrapper module (note the name of the folder containing all files is important). The module name is `my_module`.

The files in the modules are:
- `gdextension_static_library_loader.h` - this is a copy paste from `gdextension_library_loader.h` from godot source code. It enables loading static libraries instead of dynamic ones.
- `gdextension_static_library_loader.cpp` - this is a copy paste from `gdextension_library_loader.cpp` from godot source code. It enables loading static libraries instead of dynamic ones.
- `register_types.h` - This is the wrapper module you are creating. Change `my_module` and `MY_MODULE` with your module name.
- `register_types.cpp` - This is the wrapper module. You need to call your function from the gdextension. Replace `my_addon_library_init` with the function you defined in the `gdextension` in `entry_symbol` and `my_module` with your module name (this name will appear in Godot somewhere).
- `SCsub` - This file is used for Godot Module building. The `get_suffix` function replicates what `godot-cpp` does with suffixes. This loads 2 static libs. Godot-cpp and your library.


# Build your library statically (examples)

In order to build your library statically, add the following to your `SConstruct` where you already build dynamic lib:

eg.

```python
# Dynamic lib
if env["platform"] == "macos" or env["platform"] == "ios":
	library = env.SharedLibrary(
		"bin/addons/godot_sandbox/bin/libgodot_riscv{}.framework/libgodot_riscv{}".format(
			env["suffix"], env["suffix"]
		),
		source=sources,
	)
else:
	library = env.SharedLibrary(
		"bin/addons/godot_sandbox/bin/libgodot_riscv{}{}".format(env["suffix"], env["SHLIBSUFFIX"]),
		source=sources,
	)
Default(library)
# Static lib
if env["platform"] == "macos" or env["platform"] == "ios":
	library = env.StaticLibrary(
		"bin/addons/godot_sandbox/bin/libgodot_riscv{}.framework/libgodot_riscv{}{}".format(
			env["suffix"], env["suffix"], env["LIBSUFFIX"]
		),
		source=sources,
	)
else:
	library = env.StaticLibrary(
		"bin/addons/godot_sandbox/bin/libgodot_riscv{}{}".format(env["suffix"], env["LIBSUFFIX"]),
		source=sources,
	)
Default(library)
```

Here we are building both static and dynamic lib. Then in the github actions export stage, we separate the two and upload static to another folder and dynamic to another folder:

```yaml
- name: Upload Artifact
uses: actions/upload-artifact@v4
with:
    name: godot-sandbox-linux-${{ matrix.arch }}-${{ matrix.precision }}
    path: |
    ${{ github.workspace }}/bin/**/*.so
    if-no-files-found: error  

- name: Upload Artifact Static
uses: actions/upload-artifact@v4
with:
    name: static-sandbox-linux-${{ matrix.arch }}-${{ matrix.precision }}
    path: |
    ${{ github.workspace }}/bin/**/*.a
    ${{ github.workspace }}/ext/godot-cpp/**/*.a
    if-no-files-found: error  
```

Then, at the end, you package everything with the my_module wrapper. So you would distribute a module with the `godot-cpp` static lib and your library built statically. Example of this can be found in [godot-sandbox](https://github.com/libriscv/godot-sandbox).

Eg. you copy your static lib and godot-cpp static into the `my_module` folder. As help, I put `godot-cpp` destination folder in the `ext` folder, and put a `godot-cpp-static-libs.txt` here to know where to place them. I also put in the `my_library` folder a file called `my_static_lib.txt` to see where to put your static libs.

# Building the module

Then, you copy the `my_module` folder at the end to the godot `modules` folder. Then you run the build command, enabling your module:

```sh
scons module_sandbox_enabled=True target=editor
```

And that's it! You now have a module wrapper that installs an addon.

# Icons

If you have icons in your library, you need to simply make a `icons` folder and place the icons there.

# Credits

I got help designing and doing this from David Snopek (dsnopek), K. S. Ernest (iFire) Lee, Alf-André Walla (fwsGonzo).
