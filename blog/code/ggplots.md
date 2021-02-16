## My notes to plot neat ggplot figures

### Libraries 
    ggplot2
    reshape2
    dplyr

### Colors
Good colors:
    
    cbPalette <- c("#999999", "#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7")

When using continuous scale:
    
    scale_fill_gradientn(colors=c("#FFFFFF","#FFFFFF","#33DDFF","#006699","#003333","#000000"))

When using dicrete scale:

    group.colors = c("discete_value1" = cbPalette[1],"discete_value2" = cbPalette[3],"discete_value3" = cbPalette[6])


### Pre-proccessing data



When numeric values are not treated as one when plotting (For example in errors such as 'descrete value supplied to scale_x_continuous()'): 

Prepare new column as:
    
    data$new_column = as.numeric(as.character(data$column))
    
This doesn't work when only using 

    data$new_column = as.numeric(data$column)



### Scatter plots

A standard scatter plot:

    scatter  = ggplot(data=data,aes(x=value1,y=value2))  
      #Add alpha and size based on preference
      +geom_point(alpha=0.3,size=0.7) 
      
      #For numeric data, use the following. Adjust the limits once plotted.
      #Use scale_x_discrete(expand=c(0,0)) if x values aren't numeric.  
      +scale_x_continuous(expand=c(0,0),limits=c(min(data$value1),max(data$value1)))
      +scale_y_continuous(expand=c(0,0),limits=c(min(data$value2),max(data$value2)))
      
      #Based on my aestheticity
      +theme(   legend.position='bottom',legend.key = element_blank(), 
                strip.background = element_rect(color="#FFFFFF",fill="#FFFFFF"),
                panel.grid.minor = element_blank(),panel.grid.major = element_blank(),
                panel.spacing = unit(0.75, "lines"),panel.background = element_blank(),
                aspect.ratio=1.0,axis.ticks =element_line(color = "#000000", size = 0.15),
                axis.text.x = element_text(color="#000000",size=10),axis.text.y = element_text(color="#000000",size=10), 
                panel.border = element_rect(color="#000000", fill = NA)
             )
             
{need to add actuall figure here} 

    
If you have multiple replicates, compute mean and standard deviation as:

    dat = data %>% 
    group_by(list,of,columns,you,donot,want,to,group) %>% 
    summarize(mean_of_column1= mean(column),std_of_column1 = sd(column1))

{need to add actuall figure here} 


Add errorbars as:

    scatter 
      + geom_errorbar(aes(ymin =value2_mean - value2_std ,ymax = value2_mean + value2_std ))

{need to add actuall figure here} 

If both x and y axis need errorbars use '''geom_errorbarh()''' as:

    scatter
      + geom_errorbar(aes(ymin =value2_mean - value2_std ,ymax = value2_mean + value2_std ))
      + geom_errorbarh(aes(ymin =value1_mean - value1_std ,ymax = value1_mean + value1_std ))

{need to add actuall figure here} 


#### Ploting multiple 
    
    scatter
    + facet_wrap(~Condition, nrow=2)
    
    scatter
    + facet_grid(Condition~variable,scales="free_x")

Adjust the spacing between panels by changing the variable below in 'theme()' above.

    panel.spacing = unit(0.75, "lines")

{need to add actuall figure here} 


#### Trend line
    formula = y~x
    
    scatter 
    +       geom_smooth(method = "lm", formula = formula, se=FALSE)

{need to add actuall figure here} 


Statistics

{need to add actuall figure here} 

### Heatmaps

    heatmap = ggplot(dat )
    +   geom_tile(aes(X,-Y,fill = Value))  + theme_bw()  
    + scale_x_continuous(expand=c(0,0),position = "top",breaks = seq(1.25,length(unique(dat$X))+1,1) ,labels = x_labs) 
    + scale_y_continuous(expand=c(0,0),breaks = seq(-1.25,-(length(unique(dat$Y))+1),-1) ,labels = x_labs)
    
    +scale_fill_gradientn(colors=c("#FFFFFF”,"#ADD8E6","#0000FF","#003366","#000000"),limits=c(0,max(dat$Value)*1.1))
    
    +ylab("DHFR F[3] Strains")
    +xlab("DHFR F[1,2] Strains") 
    
    + theme(legend.text.align = 0,
            legend.direction = "vertical",
            legend.position = "right", 
            strip.background = element_rect(colour="#FFFFFF", color="#FFFFFF",fill="#FFFFFF"),
            panel.grid.minor = element_blank(),
            panel.grid.major = element_blank(),
            panel.background = element_blank(),
            aspect.ratio=length(unique(dat$Y))/length(unique(dat$X)),
            axis.ticks = element_blank(),
            axis.text.x = element_text(angle = 90,vjust = 0, hjust=0,size=14,color="#000000"),
            axis.text.y = element_text(angle = 0, vjust = 0, hjust=0,size=14,color="#000000"),
            axis.title.x = element_text(size = 14,color="#000000"),axis.title.y = element_text(size = 14, color="#000000")
            )
    + guides(   fill = guide_colorbar( barwidth   = 0.5, 
                                       barheight = length(unique(dat$Y))*0.75, 
                                       title="Value", 
                                       ticks =TRUE, 
                                       ticks.colour=“#000000”,
                                       ticks.linewidth =1 , 
                                       draw.ulim = TRUE, 
                                       draw.llim = TRUE , 
                                       frame.colour = “black”, 
                                       frame.linewidth=0.35)
             )
