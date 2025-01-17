# Warnings to line excerpts

There are two kinds of warnings generated by check-spelling:
1. References to a specific short string (typically a misspelled word)
2. Hits where the string itself might be unwieldy so the information is for the suggestion and not the match itself (typically a candidate pattern)

## Input

### [v0.0.21](https://github.com/check-spelling/check-spelling/releases/tag/v0.0.21) and later

```sh
Warning: samples/client/petstore/java/okhttp-gson-parcelableModel/src/main/java/org/openapitools/client/model/FileSchemaTestClass.java:222:21 ... 30, Warning - `Arrayfiles` is not a recognized word. (unrecognized-spelling)
```

### Earlier
```sh
Warning: samples/client/petstore/java/okhttp-gson-parcelableModel/src/main/java/org/openapitools/client/model/FileSchemaTestClass.java: line 222, columns 21-30, Warning - `Arrayfiles` is not a recognized word. (unrecognized-spelling)
```

## Script

Run this script passing your logs as input or a file:

```perl
#!/usr/bin/env perl

sub show_context {
    my ($show_context, $file, $line, $start, $end) = @_;
    return unless -e $file;
    open FILE, '<', $file;
    while (<FILE>) {
        next if $. < $line;
        --$start;
        --$end;
        my $context = '.' x $start;
        my $range = '.' x ($end - $start);
        if ($show_context) {
            s/($context)($range)/$1\e[31m\e[1m$2\e[0m/;
            print "$file:$.: $_";
        } else {
            s/$context($range).*/$1/;
            print "$_";
        }
        last;
    }
    close FILE;
}

sub find_context {
    my ($show_context, $file, $line, $needle) = @_;
    return unless -e $file;
    open FILE, '<', $file;
    while (<FILE>) {
        next if $. < $line;
        if ($show_context) {
            s/($needle)/\e[31m\e[1m$1\e[0m/;
            print "$file:$.: $_";
        } else {
            s/^.*($needle).*/$1/;
            print "$_";
        }
        last;
    }
    close FILE;
}

my $show_context = 0;
while (<>) {
    my ($file, $line, $start, $end, $needle);
    if (m{(?:\[|)(?:[Nn]otice|[Ww]arning|[Ee]rror)(?:: |\])([^:]+):(\d+):(\d+) \.\.\. (\d+),}) {
        ($file, $line, $start, $end) = ($1, $2, $3, $4);
    } elsif (m{Warning: ([^:]+): line (\d+), columns (\d+)-(\d+)}) {
        ($file, $line, $start, $end) = ($1, $2, $3, $4);
    } elsif (m{Warning: ([^:]+): line (\d+).*?- \`(.*?)\`.*}) {
        ($file, $line, $needle) = ($1, $2, $3);
    }
    if ($start) {
        show_context($show_context, $file, $line, $start, $end);
    } elsif ($needle) {
        find_context($show_context, $file, $line, $needle);
    }
}
```

If you set `$show_context = 0`, you'll get only the matching content. This is useful for determining whether a candidate pattern's matched content is worth excluding (by adding the candidate to `patterns.txt`).

### Running that command

Will yield output like this:
> samples/client/petstore/java/okhttp-gson-parcelableModel/src/main/java/org/openapitools/client/model/FileSchemaTestClass.java:222:      JsonArray json**Arrayfiles** = jsonObj.getAsJsonArray("files");

---
[FAQ](FAQ.md) | [Showcase](Showcase.md) | [Event descriptions](Event-descriptions.md) | [Configuration information](Configuration-information.md) | [Known Issues](Known-Issues.md) | [Possible features](Possible-features.md) | [Release notes](Release-notes.md) | [Helpful scripts](Helpful-scripts.md)
