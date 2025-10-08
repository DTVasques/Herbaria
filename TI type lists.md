# install packages
```js
install.packages("officer", dependencies = TRUE)
install.packages("tidyverse", dependencies = TRUE)
```

# Load required libraries
```js
library(officer)
library(tidyverse)
library(fs)
```

# 📂 Choose or specify the .txt file
```js
txt_path <- file.choose()  # Or use: txt_path <- "/path/to/file.txt"
```

# 📄 Read all lines from the text file
```js
lines <- readLines(txt_path, warn = FALSE) %>%
  str_trim() %>%
  discard(~ .x == "")  # remove empty lines
```

# 🔍 Detect the start of each specimen block (lines starting with a number)
```js
specimen_starts <- which(str_detect(lines, "^\\d+\\s"))
```

# Add an artificial end to simplify range logic
```js
specimen_starts <- c(specimen_starts, length(lines) + 1)
```

# 🧱 Extract specimen blocks
```js
specimen_blocks <- map2(
  specimen_starts[-length(specimen_starts)],
  specimen_starts[-1] - 1,
  ~ lines[.x:.y]
)
```

# 🧪 Parse each block into a row of data
```js
specimens_df <- map_dfr(specimen_blocks, function(block) {
  # First line: ID and Species
  number_species <- block[1]
  specimen_id <- str_extract(number_species, "^\\d+")
  species <- str_remove(number_species, "^\\d+\\s+")
```
  
  # Extract other fields (if present)
  ```js
  publication <- block[2] %||% NA
  type <- block[3] %||% NA
  location <- block[4] %||% NA
  collector <- block[5] %||% NA
  
  tibble(
    SpecimenID = specimen_id,
    Species = species,
    Publication = publication,
    Type = type,
    Location = location,
    Collector = collector
  )
})
```

# Create output file name with "_table" before extension
```js
file_name <- path_file(txt_path)
file_base <- path_ext_remove(file_name)
file_ext <- path_ext(file_name)
output_file_name <- paste0(file_base, "_table.csv")
output_path <- path(path_dir(txt_path), output_file_name)
```

# Add file name (without extension) as new column
```js
specimens_df <- specimens_df %>%
  mutate(source_file = file_base)
```

# Save CSV
```js
write_csv(specimens_df, output_path)
cat("✅ CSV saved to:", output_path, "\n")
```
