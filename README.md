# datavolley
An R package for reading DataVolley scouting files.

Beware, this is in an early stage of development and may do strange things. In particular, character encoding may not be well handled. Functionality is liable to change without warning.

## Installation

```R
library(devtools)
install_github("raymondben/datavolley")
```

## Example

Read the example data file bundled with the package:
```R
x <- read_dv(system.file("extdata/example_data.dvw",package="datavolley"),insert_technical_timeouts=FALSE)
summary(x)
```

    Match summary:
    Date: 2015-01-25
    League: Finale mladinke
    Teams: Braslovče (JERONČIČ ZORAN/MIHALINEC DAMIJANA)
           vs
           Nova KBM Branik (HAFNER MATJAŽ/)
    Result: 3-0 (25-16, 25-14, 25-22)
    Duration: 67 minutes


Number of serves by team:

```R
serve_idx <- find_serves(plays(x))
table(plays(x)$team[serve_idx])
```

      Braslovče Nova KBM Branik 
             74              54 


Distribution of serve run lengths:

```R
serve_run_info <- find_runs(plays(x)[serve_idx,])
table(unique(serve_run_info[,c("run_id","run_length")])$run_length)
```

     1  2  3  4  5  7  8 
    34 16  7  4  1  1  1 


Heatmap of attack rate by court position:

```R
attack_rate <- as.data.frame(xtabs(~start_zone,data=subset(plays(x),skill=="Attack")),
    stringsAsFactors=FALSE)
attack_rate$start_zone <- as.numeric(attack_rate$start_zone)
attack_rate$rate <- attack_rate$Freq/sum(attack_rate$Freq)

## plot
attack_rate <- cbind(attack_rate,ggxy(attack_rate$start_zone,type="start"))
ggplot(attack_rate,aes(x,y,fill=rate))+geom_tile()+ggcourt()+
    scale_fill_gradient2(name="Attack rate")
```

![Attack rate heatmap](./vignettes/attack_rate_heatmap.png?raw=true "attack rate heatmap")


Or using arrows to show the starting and ending zones of attacks:

```R
## tabulate attacks by starting and ending zone
attack_rate <- as.data.frame(xtabs(~start_zone+end_zone,data=subset(plays(x),skill=="Attack")),stringsAsFactors=FALSE)
attack_rate$start_zone <- as.numeric(attack_rate$start_zone)
attack_rate$end_zone <- as.numeric(attack_rate$end_zone)
attack_rate$rate <- attack_rate$Freq/sum(attack_rate$Freq)
attack_rate <- attack_rate[attack_rate$Freq>0,]
## starting x,y coordinates
temp <- ggxy(attack_rate$start_zone,type="start")
names(temp) <- c("sx","sy")
attack_rate <- cbind(attack_rate,temp)
## ending x,y coordinates
temp <- ggxy(attack_rate$end_zone,type="end")
names(temp) <- c("ex","ey")
attack_rate <- cbind(attack_rate,temp)
attack_rate <- attack_rate[order(attack_rate$rate,decreasing=TRUE),] ## plot in reverse order so largest arrows are on the bottom

p <- ggplot(attack_rate,aes(x,y,col=rate))+ggcourt()+scale_fill_gradient2(name="Attack rate")
for (n in 1:nrow(attack_rate)) p <- p+geom_path(data=data.frame(x=c(attack_rate$sx[n],attack_rate$ex[n]),y=c(attack_rate$sy[n],attack_rate$ey[n]),rate=attack_rate$rate[n]),aes(size=rate),lineend="round",arrow=arrow(ends="last",type="closed"))+guides(size="none")
p+scale_fill_gradient(name="Attack rate")
```

![Attack rate by start and end zone](./vignettes/attack_rate_arrows.png?raw=true "attack rate by start and end zone")


## Troubleshooting

If you see unexpected behaviour, try `read_dv(...,do_warn=TRUE)` to obtain more diagnostic information during the process of reading and parsing the DataVolley file. Also check the text encoding specified to `read_dv` (did you specify one??)


## More

For more information about DataVolley, see http://www.dataproject.com/IT/en/Volleyball.

