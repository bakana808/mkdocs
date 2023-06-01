##  `scan_folder(dirname, extensions)`
Returns a list of files in a directory that has the given extension.

```gdscript
static func scan_folder(dirname: String, extensions: PackedStringArray) -> PackedStringArray:
    var dir = DirAccess.open(dirname)
    var filepaths = PackedStringArray()

    if dir:
        dir.list_dir_begin()
        while true:
            var path := dir.get_next()
            if path == "": break

            if path.get_extension() in extensions:
                # items.append(LevelSelection.new(path))
                var filepath = dirname + path
                filepaths.append(filepath)

        dir.list_dir_end()

    return filepaths

# Examples:
scan_folder("levels/", ["tscn"])  # list all .tscn files in the "levels" folder
```